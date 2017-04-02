---
layout:     post
title:      "「技术」Kafka使用浅谈"
subtitle:   "Something about using Kafka with go and php"
date:       2017-03-31 21:00:00
author:     "Lucifa Diva"
header-img: "img/post-bg-apple-event-2015.jpg"
tags:
    - 技术
---


<div>
    <blockquote>先挖个坑。。</blockquote>
    <br>最近由于各种需求密集使用Kafka，同时在使用中遇到了各类问题，由简单的使用，被迫系统的研究一下。
    <br> 一、Why Kafka
    <br>    http://kafka.apache.org/documentation 首先与传统消息系统比，Kafka具有高吞吐、冗余备份、高容错性的特点，特别适用于大量的消息处理应用，同时Kafka自身设计理念就是分布式，水平扩展容易并且性能不变，写文件采用zero-copy方式几乎与内存效率等同，这个后边介绍，并且由于是写文件方式存储消息，持久化没有问题，因此对于量大的消息，或者不允许丢失的消息应用（主要是rabbitmq、redis、memcached等主要基于内存，进程崩溃消息基本丢失），用Kafka比较合适。
    <br>二、Messaging Use
    <br>    与其他消息系统一样，当用作消息系统时，我们需要关注三方面：
    <br>    (1)Kafka基本概念
    <br>    作为消息存储的载体，我们需要了解一下Kafka，单从使用者角度，我们只需知道，Kafka的存储是文件类型，首先需要清楚几个概念：
    <br>    broker:可以理解未一个Kafka实例；
    <br>    Topic:某种类型的消息，例如用户浏览行为数据存在topic1中，用户点击行为存在topic2中；
    <br>    Partition:每种topic还分为N个partition，每个partition的消息是有序的，如partition1：[msg1,msg5,msg9]，partition2:[msg2,msg3,msg8]，partition1内部各个消息是有序队列，同样partition2也一样，但是partition1和parititon2之间的消息不一定是有序的;每个partition内部消息是有一个递增的顺序号offset；
    <br>    Message：即消息体，msg1
    <br>    Producer：消息生产者，谁把这条消息发送到kafka的；
    <br>    Consumer：消息消费者，谁把这条消息从kafka拿出来使用，比如消费topic1的partition1消息，按照offset从小到大消费，即msg1->msg5->msg9；
    <br>    question1：为什么topic要划分N个partition呢？
    <br>    从特性倒推一下设计原因，不难发现每个partition内部保序，但各个partition之间不保序，这样就从逻辑上划分了保序队列，而这些保序队列就可以并行处理，互不影响了。
    <br>    question2：Producer与Consumer是一一对应的吗？
    <br>    并不是，这就是Kafka设计的高效之处，生产者生产的某个topic数据Kafka只存一份（这个一份不包括replica哟），但可能消费者有多个，假设topic1是用户点击数据消息，consumer1要用他们统计点击明细，consumer2要用他们大数据分析用户兴趣，consumer3要用他们给网站按点击评分，他们实际上使用的都是同一份数据，每隔consumer自己维护自己消费到哪条消息了，即offset；consumer1消费的比较快，已经消费到了msg9，他只需默默记下自己offset=3，下次从offset=4的消息开始消费，而consumer2比较慢，才消费到msg1，他记下自己offset=1，下次从offset=2的消息开始消费。。。所以，从逻辑上也可以看出，每个topic的消费者数量是没有限制的，是1对0/N的关系。
    <p>
    <b></b>
    </p>
</div>


