# kafka使用说明
引入kafka相关jar包：
```
<dependency>
    <groupId>com.navinfo.wecloud</groupId>
    <artifactId>wecloud-starter-kafka</artifactId>
</dependency>
```
在Application中引用注解：
```
@EnableKafkaProducer
@EnableKafkaConsumer
```
在application.properties中添加kafka配置：
```
#kafka producer config
wecloud.kafka.producer.servers=localhost:9092
wecloud.kafka.producer.keySerializerClass=org.apache.kafka.common.serialization.StringSerializer
wecloud.kafka.producer.valueSerializerClass=com.navinfo.wecloud.common.kafka.serializers.json.KafkaJsonSerializer
wecloud.kafka.producer.batchSize=131072
wecloud.kafka.producer.bufferMemory=33554432
wecloud.kafka.producer.requestTimeoutMs=30000
wecloud.kafka.producer.retries=0
wecloud.kafka.producer.lingerMs=1
wecloud.kafka.producer.compressionType=gzip

#kafka consumer config
wecloud.kafka.consumer.servers=localhost:9092
wecloud.kafka.consumer.concurrencySize=1
wecloud.kafka.consumer.topics=wecloud-kafka-demo
wecloud.kafka.consumer.sessionTimeoutMs=15000
wecloud.kafka.consumer.enableAutoCommit=true
wecloud.kafka.consumer.autoCommitIntervalMs=500
wecloud.kafka.consumer.clientId=wecloud-kafka-demo
wecloud.kafka.consumer.groupId=wecloud-kafka-demo
wecloud.kafka.consumer.keySerializerClass=org.apache.kafka.common.serialization.StringDeserializer
wecloud.kafka.consumer.valueSerializerClass=com.navinfo.wecloud.common.kafka.serializers.json.KafkaJsonDeserializer
wecloud.kafka.consumer.fetchMinBytes=40
wecloud.kafka.consumer.fetchMaxWaitMs=500
wecloud.kafka.consumer.maxPartitionFetchBytes=1048576
wecloud.kafka.consumer.ssl.protocol=TLS
wecloud.kafka.consumer.ssl.truststorePassword=
wecloud.kafka.consumer.ssl.truststoreLocation=
wecloud.kafka.consumer.ssl.keystoreLocation=
wecloud.kafka.consumer.ssl.keystorePassword=
wecloud.kafka.consumer.autoOffsetReset=latest
wecloud.kafka.consumer.service.demo_1.0=com.navinfo.wecloud.demo.kafka.KafkaConsumerHandler
```
kafka生产者和消费者代码示例如下：
```
/**
 * @Desc: 往kafka发送数据
 * @Author: ZhangYue
 * @Date: create in 2017/12/1
 * @copyright Navi WeCloud
 */
@Component
@Configurable
@EnableScheduling
public class KafkaProducer {

    public static Logger logger = LoggerFactory.getLogger(KafkaProducer.class);

    @Autowired
    private KafkaMessageChannel kafkaMessageChannel;

    @Scheduled(cron = "${kafka.send.schedule.cron:0/30 * * * * ?}")
    public void run() {
        try {
            KafkaBean bean = new KafkaBean();
            bean.setMessage("hello,I'm testing kafka!");
            Long currentTime = System.currentTimeMillis();
            bean.setTime(currentTime);
            this.sendKafka(bean);
            Long sendEndTime = System.currentTimeMillis();
            logger.info("往kafka发送数据{},耗时:{}ms", JSONObject.toJSONString(bean), sendEndTime-currentTime);
        } catch (Exception e) {
            logger.error("TA监控任务出错", e);
        }
    }

    /**
     * 发送监控报警数据
     * @param bean
     * @throws Exception
     */
    public void sendKafka(KafkaBean bean) throws Exception{
        //发送kafka消息给服务消费者
        KafkaMessage kafkaMessage = new KafkaMessage();
        kafkaMessage.setSendTime(new Date());
        kafkaMessage.setKey("testKey");
        kafkaMessage.setVersion("1.0");
        kafkaMessage.setServiceId("com/navinfo/wecloud/demo");
        kafkaMessage.setTopic("wecloud-kafka-com.navinfo.wecloud.demo");
        kafkaMessage.setMessage(JSONObject.toJSONString(bean).getBytes());
        kafkaMessageChannel.send(kafkaMessage);
    }


}

/**
 * @Desc: 消费kafka数据
 * @Author: ZhangYue
 * @Date: create in 2017/12/1
 * @copyright Navi WeCloud
 */
@Component
public class KafkaConsumerHandler implements KafkaMessageHandler<KafkaMessage> {

    private static Logger logger = LoggerFactory.getLogger(KafkaConsumerHandler.class);

    @Override
    public void handle(KafkaMessage kafkaMessage) {
        logger.info("开始消费kafka数据");
        try {
            KafkaBean kafkaBean = JSONObject.parseObject(kafkaMessage.getMessage(),KafkaBean.class);
            logger.info("消费到kafka数据{}",JSONObject.toJSONString(kafkaBean));
        } catch (Exception e) {
            e.printStackTrace();
            logger.error("消费kafka数据出现异常！");
        }
    }


}
```

