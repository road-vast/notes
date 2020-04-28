# VirtualHost











RabbitMq的VirtualHost（虚拟消息服务器），每个VirtualHost相当于一个相对独立的RabbitMQ服务器；每个VirtualHost之间是相互隔离的，exchange、queue、message不能互通。 

拿数据库（用MySQL）来类比：RabbitMq相当于MySQL，RabbitMq中的VirtualHost就相当于MySQL中的一个库。