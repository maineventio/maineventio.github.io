# Decisions
_Why we're using the technologies we're using, with links & resources_

### Architecture: Lambda
The best practice for combining fast analytics on massive historical data, with real-time changes to that data.
Check out [this key post by Amplitude](https://amplitude.com/blog/2015/08/25/scaling-analytics-at-amplitude/) on how they used this architecture to scale to billions of daily events.  They did a follow up on a new approach, but still use the Lambda architecture as the basis of the whole thing.

* [Seminal book by Marz on this architecture](https://www.amazon.com/Big-Data-Principles-practices-scalable/dp/1617290343/?&tag=rnwap-20)
* [Repository of info about Lambda](http://lambda-architecture.net/)
* [Update post by Amplitude](https://amplitude.com/blog/2016/05/25/nova-architecture-understanding-user-behavior/)

**What about Kappa?!  It's a streaming world now!**

Kappa is a simplification of Lambda that removes the batch layer, and focuses entirely on streaming.  In addition, we're seeing more of a move towards streaming-only architectures.
 * [About Kappa](http://milinda.pathirage.org/kappa-architecture.com/)
 * [Questioning the Lambda architecture](https://www.oreilly.com/ideas/questioning-the-lambda-architecture)
 
One of the most important critiques of Lambda is that it can require coding different approaches to the same summary jobs on different platforms, like Hadoop and Storm.  Under Spark, this is not a real issue.

**What about pure streaming?**

Likewise, some newer approaches that focus on real-time streams (vs Spark's near-real-time microbatches) like Apache Flink.  They look very interesting, and it's extremely possible that our architecture will evolve as more is learned about these, their issues, their strengths, and how they can fit in.
 
Today, Lambda on Spark is well-proven and will be a dependable, scalable, and cost-efficient solution that reaches our performance goals.

* [The world beyond batch: Streaming 101](https://www.oreilly.com/ideas/the-world-beyond-batch-streaming-101)
* [Apache Flink: A new wave to real-time stream processing](https://dzone.com/articles/packaging-apache-flink-applications-with-dependenc)


### Data Platform: Spark
Hadoop and MapReduce are the old way, Spark is tailor-made for the Lambda architecture.  It provides a consistent programming interface using Scala that can be run on both the batch data and the real-time streaming data.

> "the biggest advantage Spark gave us in this case was Spark Streaming, which allowed us to re-use the same aggregates we wrote for our batch application on a real-time data stream." 

* [Lambda Architecture with Apache Spark](https://dzone.com/articles/lambda-architecture-with-apache-spark)
* [Talk: Implementing Lambda with Spark](https://www.youtube.com/watch?v=MbJVij3im4c&t=2103s)
* [Spark & Kafka: Achieving zero data-loss](http://aseigneurin.github.io/2016/05/07/spark-kafka-achieving-zero-data-loss.html)
* [Real-time stream processing using Apache Spark Streaming and Kafka on AWS](https://aws.amazon.com/blogs/big-data/real-time-stream-processing-using-apache-spark-streaming-and-apache-kafka-on-aws/)
* [Building Lambda architecture with Spark Streaming (Cloudera)](http://blog.cloudera.com/blog/2014/08/building-lambda-architecture-with-spark-streaming/)

### Message Broker: Kafka
Kafka is "fast, durable, scalable and easy to operate". Even compared to AWS solutions SQS and Kinesis, OpsClarity (below) found it "blazingly fast", "durable", and "easy to scale" and has a cost advantage over the AWS services.

Another post (below) comparing Kafka with other MQ products, determined "offers a high guarantee that the service will be available and non-blocking under any circumstances. In addition, messages can easily be replicated for higher data availability. Kafka performance is just great and resource usage modest".

Overall, Kafka seems like the professional, scalable choice and it is the queue of choice in many implementations of Spark, so we'll be using it.

* [Kafka vs Kinesis vs SQS](https://www.opsclarity.com/evaluating-message-brokers-kafka-vs-kinesis-vs-sqs/)
* [Exploring Message Brokers](https://www.percona.com/blog/2014/05/05/exploring-message-brokers/)
* [Kafka](https://kafka.apache.org/)

### Storage: S3 and Minio
Using an S3-compatible storage for long term immutable data seems like a no-brainer.  It's been such a successful platform that there are plenty of options that support the API for on-prem installation, and anyone already using AWS or S3 can BYO-bucket.

For our local and containerized depoyments, I've chosen Minio just because it seems to have the most traction of the open-source alternatives.  

The nice thing about this area is, if we don't like Minio we can swap out that component without altering the integrations.
* [Article on Minio](http://www.theregister.co.uk/2016/12/21/minio_microserever_aims_for_object_world_domination/)
* [Minio GitHub](https://github.com/minio/minio)

_In many use cases, we could also just rely on existing HDFS clusters_

### Dashboard, project management, reporting: Laravel
Laravel is a dream to develop in, has a huge active community and a mammoth library available thanks to Symfony.  For standard tasks like project creation, team invitations, etc... we will be relying on Laravel.  

One of the biggest benefits here is it's very easy for any client or user/developer to customize the behavior and functionality.  Change the authentication providers, tweak the reports, etc...

* [Laravel](https://laravel.com/)
* [Laracasts: Learning laravel's ecosystem](https://laracasts.com/)

### API Endpoint: Laravel Lumen
It may be faster to use Node, or Go.  But we already have multiple languages in this platform, thanks to the Scala used on Spark and the commitment to PHP Laravel in the dashboard, so let's keep the other functional areas that people may customize to PHP.

Lumen is Laravel's lighter, faster sister, and this is definitely an area open to customization - whether authentication or filtering, I expect client developers to be modifying the endpoint code. 

So KISS.

Very easy to create alternatives to this, since the main function of this area is to:
1. Validate project ID, sender, etc... as necessary
2. Push the event payload to a queue
3. Repeat at volume.

I'm not married to the Lumen approach, I have suspiciouns about Taylor's long term commitment to it and it's not exactly a vibrant community.  But doing this part of the work in ANYTHIGN ELSE is pretty easy in the future.  

* [Lumen](https://lumen.laravel.com/)
* [Introducing Lumen](https://mattstauffer.co/blog/introducing-lumen-from-laravel)
* [Lumen performance in production](http://www.darwinbiler.com/laravel-lumen-performance-production/)

### Web Interpreter: PHP-FPM, PHP7
HHVM may be faster in some situations, but for this project speed of development and fewest possible moving parts (already quite complex) is key, so we're going to stay un-fancy and use PHP7 and PHP-FPM instead of worrying about HHVM compatibility.
Overall,
* both are very fast
* PHP7 requires no learning curve or customization
* PHP7 long term support is guaranteed, HHVM is not.

YMMV, love to see someone try it out :)

* [PHP7 vs HHVM](https://www.keycdn.com/blog/php-7-vs-hhvm/)

### Querying & Combining: Druid
Druid in fact gives us the kind of distributed column-store system for rapid interactive queries that Amplitude developed [in their 2016 follow-up post](https://amplitude.com/blog/2016/05/25/nova-architecture-understanding-user-behavior/), and appears to be the perfect system that will bridge the real-time ingestion and state manegement of Spark to the fast queries of user-facing reporting.  And it comes with a Spark plugin for just such an occasion.
> Druid's focus is on extremely low latency queries, and is ideal for powering applications used by thousands of users, and where each query must return fast enough such that users can interactively explore through data.

* [Druid](http://druid.io/)
* [Combining Druid and Spark: Interactive and flexible analytics at scale](https://www.linkedin.com/pulse/combining-druid-spark-interactive-flexible-analytics-scale-butani)
* [Druid vs Spark](http://druid.io/docs/latest/comparisons/druid-vs-spark.html)

*Also check out: https://github.com/filodb/FiloDB*

### Reporting
Things to check out:
* [Apache Zeppelin](https://zeppelin.apache.org/)
* [AirBnb Superset](https://github.com/airbnb/superset)
* [Metabase](https://github.com/metabase/metabase)

