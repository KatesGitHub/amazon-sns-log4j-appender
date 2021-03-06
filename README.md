Introduction
============

There are some errors that are just too important to miss. [Amazon SNS](http://aws.amazon.com/sns/)
is a very easy-to-use PubSub service that can send you e-mails or texts (or be
added to an SQS queue so you can do your own processing on it). This log4j
appender lets you easily set a threshold past which logged errors will be sent
to an SNS topic, all with 0 additional lines of code (and a couple of lines of
properties files).

Installation
------------

amazon-sns-log4j-appender is built and distributed with Maven. You can include
it in your project with the following dependency in your `pom.xml` file:

    <dependency>
        <groupId>com.twitsprout</groupId>
        <artifactId>amazon-sns-log4j-apender</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>

It is not currently deployed to the central repository, so you must also
add the [TwitSprout](http://twitsprout.com) Maven repository to the
`repositories` section of your `pom.xml` file as well:

    <repository>
      <id>cloudbees-twitsprout-release-repository</id>
      <url>https://repository-twitsprout.forge.cloudbees.com/release/</url>
    </repository>

Usage
-----

You don't need any extra code to use this appender; you just need two pieces of
configuration:

  1. An `AwsCredentials.properties` file following the regular format of the
     AWS SDK for Java, described [here](http://aws.amazon.com/articles/3586).
  2. Add an appender in your `log4j.properties` file using the class
     `com.twitsprout.appender.sns.SnsAsyncAppender`. Make sure to set the
     `TopicName` and `Threshold` properties on the appender as well.

You can find a fully-working sample (including `log4j.properties.sample`) in
 `src/test`. Just provide your own AWS credentials.

FAQ
---

  **Do I need to wrap this appender in log4j's AsyncAppender?**

  I don't think it will hurt, but it's not necessary. Even as a regular
  appender, it uses the Async client for SNS, so your log statements won't
  block while they get sent. It's really only the penalty for creating the
  request object that you'll be incurring directly, which shouldn't be an
  issue since this appender should only be used for **critical** errors.

  **Why is my log message being truncated?**

  Amazon SNS limits published messages to 64K. If your log message exceeds that
  amount, the appender will automatically truncate it. Keep these messages short
  and sweet, just enough info for your admin in the field to know what's going on
  before he finds a computer to do some real analysis.

  **Why am I getting strange `NoClassDefFoundError`s all over the place?**

  Make sure you have both Log4J and the AWS SDK for Java already added to your
  classpath. The POM sets these dependencies to `<scope>provided</scope>` to keep
  it compatible with custom versions of both of these libraries which many
  sophisticated users will often want to use.

  **Why does using this appender completely crash my process, using up
  truly ridiculous amounts of CPU?**

  The Threshold should certainly be set higher than `INFO`, otherwise the logging
  messages produced by the AWS SDK's own HTTP client will overwhelm the logger.

