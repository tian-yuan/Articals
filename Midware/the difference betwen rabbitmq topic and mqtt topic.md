### the difference between rabbitmq topic and mqtt topic

#### the topic of rabbitmq
The routing pattern follows the same rules as the routing key with the addition 
that * matches a single word, and # matches zero or more words.
Thus the routing pattern *.stock.# matches the routing keys usd.stock 
and eur.stock.db but not stock.nasdaq.

*.stock.#

. : topic separator
* : matches a single word
# : matches zero or more words

#### the topic of mqtt topic

myhome/+/livingroom/#
/ : topic separator
+ : matches a single topic level
# : matches an arbitrary number of topic levels, zero or more topic levels


$SYS/broker/clients/connected
$ : will treated specified

### conclusion
so the rabbitmq topic is same to mqtt topic essentially

we can use https://github.com/davedoesdev/qlobber in the mqtt topic project,
which change the '/' to '.' and change the '*' to '+'

