
Cache和Buffer是两个不同的概念，简单的说，Cache是加速“读”，而buffer是缓冲“写”，前者解决读的问题，保存从磁盘上读出的数据，后者是解决写的问题，保存即将要写入到磁盘上的数据。在很多情况下，这两个名词并没有严格区分，常常把读写混合类型称为buffer cache，本文后续的论述中，统一称为cache。

Oracle中的log buffer是解决redo写入的问题，而data buffer cache则解决data block的读写问题。对于Oracle来说，如果IO没有在SGA中命中，都会发生物理IO，Oracle并不关心底层存储的类型，可能是一套存储系统，可能是本地磁盘，可能是RAID 10，也可能是RAID 5，可能是文件系统，也可能是裸设备，或是ASM。总之，Oracle把底层的存储系统称为存储子系统。

在存储系统中，cache几乎无处不在（在后面的论述中，我们统称为cache），文件系统有cache，存储有cache，RAID控制器上有cache，磁盘上也有cache。为了提高性能，Oracle的一个写操作，很有可能写在存储的cache上就返回了，如果这时存储系统发生问题，Oracle如何来保证数据一致性的问题。

Oracle数据库最重要的特性是：Write ahead logging，在data block在写入前，必须保证首先写入redo log，在事务commit时，同时必须保证redo log被写入。Oracle为了保证数据的一致性，对于redo log采用了direct IO，Direct IO会跳过了OS上文件系统的cache这一层。但是，OS管不了存储这一层，虽然跳过了文件系统的cache，但是依然可能写在存储的cache上。

一般的存储都有cache，为了提高性能，写操作在cache上完成就返回给OS了，我们称这种写操作为write back，为了保证掉电时cache中的内容不会丢失，存储都有电池保护，这些电池可以供存储在掉电后工作一定时间，保证cache中的数据被刷入磁盘，不会丢失。不同于UPS，电池能够支撑的时间很短，一般都在30分钟以内，只要保证cache中的数据被写入就可以了。存储可以关闭写cache，这时所有的写操作必须写入到磁盘才返回，我们称这种写操作为write throuogh，当存储发现某些部件不正常时，存储会自动关闭写cache，这时写性能会下降。

RAID卡上也有cache，一般是256M，同样是通过电池来保护的，不同于存储的是，这个电池并不保证数据可以被写入到磁盘上，而是为cache供电以保护数据不丢失，一般可以支撑几天的时间。还有些RAID卡上有flash cache，掉电后可以将cache中的内容写入到flash cache中，保证数据不丢失。如果你的数据库没有存储，而是放在普通PC机的本地硬盘之上的，一定要确认主机中的RAID卡是否有电池，很多硬件提供商默认是不配置电池的。当然，RAID卡上的cache同样可以选择关闭。

磁盘上的cache，一般是16M-64M，很多存储厂商都明确表示，存储中磁盘的cache是禁用的，这也是可以理解的，为了保证数据可靠性，而存储本身又提供了非常大的cache，相比较而言，磁盘上的cache就不再那么重要。SCSI指令中有一个FUA(Force Unit Access)的参数，设置这个参数时，写操作必须在磁盘上完成才可以返回，相当于禁用了磁盘的写cache。虽然没有查证到资料，但是我个人认为一旦磁盘被接入到RAID控制器中，写cache就会被禁用，这也是为了数据可靠性的考虑，我相信存储厂商应该会考虑这个问题。

至此，我们可以看到Oracle的一个物理IO是经历了一系列的cache之后，最终被写入到磁盘上。cache虽然可以提高性能，但是也要考虑掉电保护的问题。关于数据的一致性，是由Oracle数据库，操作系统和存储子系统共同来保证的。

文章来源：http://ourmysql.com/archives/859
