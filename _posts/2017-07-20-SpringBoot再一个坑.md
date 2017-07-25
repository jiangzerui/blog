---
layout: post
title: SpringCloud 再一个坑
date: 2017-07-120
categories: 工作
---

决定要把微服务的架构搞明白，同时用一个项目来验证这套架构是否符合构建一个完整的微服务体系。

这段时间很受伤，经常熬到半夜穿着衣服就睡着了，严重拖延了我的金融学习...

不过既然要搞明白，就必须一个一个坑的过。

纪录一下今天的坑：

异常，永远都从Exception开始，不过有异常是好的，最怕是狗屁不出来...

废话不说，描述异常。

feign.RetryableException: connect timed out executing GET http://sc/users/] with root cause

因为用了feign进行业务与前台的脱离，晚上回家稍事休息后打开久久未开的电脑，从消费者调用service居然异常了。在网上查到的解释是：


        feign client的retry及超时设置


        默认值
        默认maxAttempts值
        /Users/xixicat/.m2/repository/io/github/openfeign/feign-core/9.3.1/feign-core-9.3.1-sources.jar!/feign/Retryer.java

        public Default() {
              this(100, SECONDS.toMillis(1), 5);
            }

            public Default(long period, long maxPeriod, int maxAttempts) {
              this.period = period;
              this.maxPeriod = maxPeriod;
              this.maxAttempts = maxAttempts;
              this.attempt = 1;
            }
        retry逻辑
        /Users/xixicat/.m2/repository/io/github/openfeign/feign-core/9.3.1/feign-core-9.3.1-sources.jar!/feign/Retryer.java

        public void continueOrPropagate(RetryableException e) {
              if (attempt++ >= maxAttempts) {
                throw e;
              }

              long interval;
              if (e.retryAfter() != null) {
                interval = e.retryAfter().getTime() - currentTimeMillis();
                if (interval > maxPeriod) {
                  interval = maxPeriod;
                }
                if (interval < 0) {
                  return;
                }
              } else {
                interval = nextMaxInterval();
              }
              try {
                Thread.sleep(interval);
              } catch (InterruptedException ignored) {
                Thread.currentThread().interrupt();
              }
              sleptForMillis += interval;
            }
        这里attempt初始值为1，即把第一次的请求也算上去了，先执行attempt >= maxAttempts判断，再执行attempt++。因此maxAttempts设置为2表示重试1次。/Users/xixicat/.m2/repository/io/github/openfeign/feign-core/9.3.1/feign-core-9.3.1-sources.jar!/feign/SynchronousMethodHandler.java

        @Override
          public Object invoke(Object[] argv) throws Throwable {
            RequestTemplate template = buildTemplateFromArgs.create(argv);
            Retryer retryer = this.retryer.clone();
            while (true) {
              try {
                return executeAndDecode(template);
              } catch (RetryableException e) {
                retryer.continueOrPropagate(e);
                if (logLevel != Logger.Level.NONE) {
                  logger.logRetry(metadata.configKey(), logLevel);
                }
                continue;
              }
            }
          }
        关于++
            @Test
            public void testPlus(){
                int attempt = 1;
                int maxAttempts = 1;
                try{
                    if(attempt++ == maxAttempts){
                        throw new RuntimeException("EXCEED");
                    }
                }finally {
                    System.out.println(attempt);
                }
            }
        输出

        2

        java.lang.RuntimeException: EXCEED

          at XXXTest.testPlus(XXXTest.java:50)
        默认超时时间
        /Users/xixicat/.m2/repository/io/github/openfeign/feign-core/9.3.1/feign-core-9.3.1-sources.jar!/feign/Request.java

            public Options(int connectTimeoutMillis, int readTimeoutMillis) {
              this.connectTimeoutMillis = connectTimeoutMillis;
              this.readTimeoutMillis = readTimeoutMillis;
            }

            public Options() {
              this(10 * 1000, 60 * 1000);
            }
        参数设置
        timeout设置
            @Bean
            Request.Options feignOptions() {
                return new Request.Options(/**connectTimeoutMillis**/1 * 1000, /** readTimeoutMillis **/1 * 1000);
            }
        retry配置
            @Bean
            Retryer feignRetryer() {
                return Retryer.NEVER_RETRY;
            }
        响应时间
        默认retry响应时间(1s connectTimeout,1s readTimeout)
        Percentage of the requests served within a certain time (ms)
          50%   6718
          66%   7020
          75%   7371
          80%   8237
          90%   8404
          95%   8404
          98%   8404
          99%   8404
         100%   8404 (longest request)
        不retry(1s connectTimeout,1s readTimeout)
        Percentage of the requests served within a certain time (ms)
          50%   1219
          66%   1230
          75%   1307
          80%   1485
          90%   1674
          95%   1674
          98%   1674
          99%   1674
         100%   1674 (longest request)
        不retry(5s connectTimeout,5s readTimeout)
        Percentage of the requests served within a certain time (ms)
          50%   5561
          66%   5592
          75%   5653
          80%   5677
          90%   5778
          95%   5778
          98%   5778
          99%   5778
         100%   5778 (longest request)
        小结
        feign client默认的connectTimeout为10s，readTimeout为60.单纯设置timeout，可能没法立马见效，因为默认的retry为5次.因此，如果期望fail fast的话，需要同时自定义timeout以及retry的参数，而且要确保反向代理，比如nginx的proxy_connect_timeout以及proxy_read_timeout要大于feign的配置才能见效，不然对外部用户感知到的还是nginx的504 Gateway Time-out，起不到fallback的效果。


当前在开发模式，没有按照这个进行修改，重启好了。不过先收藏了，等了生产库的时候在进行调试。

当前最重要完成项目！


再来一个解释：

<img src="{{ site.baseurl }}/public/asset/pic/exception01.png" style="width:60%;display:inline;text-align: center;" />


import feign.Request;

@Configuration
@EnableDiscoveryClient
@EnableFeignClients(basePackageClasses = { ServiceFeignClient.class })
@ComponentScan(basePackageClasses = { ServiceFeignClient.class })
public class FeignConfig {

    /**
     * Method to create a bean to increase the timeout value,
     * It is used to overcome the Retryable exception while invoking the feign client.
     * @param env,
     *            An {@link ConfigurableEnvironment}
     * @return A {@link Request}
     */
    @Bean
    public static Request.Options requestOptions(ConfigurableEnvironment env) {
        int ribbonReadTimeout = env.getProperty("ribbon.ReadTimeout", int.class, 70000);
        int ribbonConnectionTimeout = env.getProperty("ribbon.ConnectTimeout", int.class, 60000);

        return new Request.Options(ribbonConnectionTimeout, ribbonReadTimeout);
    }
}
