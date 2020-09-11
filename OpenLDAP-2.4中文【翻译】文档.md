# # OpenLDAP-2.4中文【翻译】文档

OpenLDAP Software 2.4 Administrator's Guide

> 此文档方便自己翻阅，使用Typora体验更佳
> 原文地址
> https://www.openldap.org/doc/admin24/guide.html

##  1. OpenLDAP目录服务简介

本文档描述了如何构建、配置和操作OpenLDAP软件以提供目录服务。这包括关于如何配置和运行独立LDAP守护进程slapd(8)的详细信息。它是为新的和有经验的行政人员准备的。本节提供目录服务的基本介绍，特别是slapd(8)提供的目录服务。本介绍仅提供足够的信息，以便您可以开始了解LDAP、X.500和目录服务。



### 1.1. 什么是目录服务?

目录是专门为搜索和浏览而设计的专门数据库，还支持基本的查找和更新功能。

> 注意:有些人将目录定义为只读访问优化的数据库。这个定义充其量是过于简单化了。

目录往往包含描述性的、基于属性的信息，并支持复杂的过滤功能。目录通常不支持为处理大容量复杂更新而设计的数据库管理系统中的复杂事务或回滚方案。目录更新通常是简单的全有或全无更改(如果允许的话)。目录通常经过调优，以对大量查找或搜索操作提供快速响应。它们可能具有广泛复制信息的能力，以提高可用性和可靠性，同时减少响应时间。在复制目录信息时，只要及时解决了不一致，使用者之间暂时的不一致就没有问题。

提供目录服务有许多不同的方法。不同的方法允许不同种类的信息存储在目录中，对信息如何被引用、查询和更新，以及如何防止未经授权的访问等有不同的要求。有些目录服务是本地的，为受限制的上下文提供服务(例如，在一台机器上的finger服务)。其他服务是全球性的，为更广泛的环境(例如，整个互联网)提供服务。全局服务通常是分布式的，这意味着它们包含的数据分布在许多机器上，所有这些机器协作提供目录服务。通常，全局服务定义一个统一的名称空间，该名称空间提供相同的数据视图，而不管您在数据本身的哪个位置。

web目录，例如Curlie Project &lt;https://curlie.org>提供的目录，是目录服务的一个很好的例子。这些服务对web页面进行编目，并专门为支持浏览和搜索而设计。

虽然有些人认为互联网域名系统(DNS)是全球分布式目录服务的一个例子，但DNS是不能浏览或搜索的。将其描述为全局分布式查找服务更恰当。

###  1.2 LDAP是什么?

LDAP代表轻量级目录访问协议。顾名思义，它是一种轻量级协议，用于访问目录服务，特别是基于x .500的目录服务。LDAP在TCP/IP或其他面向连接的传输服务上运行。LDAP是IETF标准跟踪协议，在RFC4510的“轻量级目录访问协议(LDAP)技术规范路线图”中指定。

本节从用户的角度概述LDAP。

目录中可以存储哪些类型的信息?LDAP信息模型基于条目。条目是具有全局惟一的专有名称(DN)的属性集合。DN用于明确地引用条目。条目的每个属性都有一个类型和一个或多个值。这些类型通常是助记符字符串，比如“cn”表示通用名称，“mail”表示电子邮件地址。值的语法取决于属性类型。例如，cn属性可能包含Babs Jensen值。邮件属性可能包含值“babs@example.com”。jpegPhoto属性将包含JPEG(二进制)格式的照片。

信息是如何安排的?在LDAP中，目录条目以类似树的层次结构排列。传统上，这种结构反映了地理和/或组织边界。表示国家的条目出现在树的顶部。下面是代表国家和国家组织的条目。下面可能是代表组织单位、人员、打印机、文档或您能想到的任何东西的条目。图1.1显示了使用传统命名的示例LDAP目录树。

```
图1.1 <Figure 1.1-LDAP directory tree (traditional naming).png>
```

该树也可以基于互联网域名进行安排。这种命名方法越来越流行，因为它允许使用DNS定位目录服务。图1.2显示了使用基于域命名的示例LDAP目录树。

```
图1.2 <Figure 1.2 - LDAP directory tree (Internet naming).png>
```

此外，LDAP允许您通过使用称为objectClass的特殊属性来控制条目中需要和允许哪些属性。objectClass属性的值确定条目必须遵守的模式规则。

这些信息是如何被引用的?条目通过其专有名称来引用，该专有名称是通过获取条目本身的名称(称为相对专有名称或RDN)并连接其祖先条目的名称来构造的。例如，在上面的互联网命名示例中Barbara Jensen条目的RDN为uid=babs, DN为uid=babs,ou=People,dc=example,dc=com。完整的DN格式在RFC4514“LDAP:专有名称的字符串表示”中有描述。

如何访问信息?LDAP定义了查询和更新目录的操作。提供了从目录中添加和删除条目、更改现有条目以及更改条目名称的操作。但是，大多数时候使用LDAP搜索目录中的信息。LDAP搜索操作允许搜索目录的某些部分，以查找与搜索筛选器指定的某些条件匹配的条目。可以从符合条件的每个条目中请求信息。

如，您可能想搜索dc=example,dc=com处及其下面的整个目录子树，查找名为Barbara Jensen的人，检索找到的每个条目的电子邮件地址。LDAP允许您轻松地完成此操作。或者您可能想搜索st=California,c=US条目正下方的条目，以查找名称中带有字符串Acme且具有传真号的组织。LDAP也允许您这样做。下一节将更详细地描述使用LDAP可以做什么，以及它对您的有用之处。

如何保护资料不受未经授权的访问?有些目录服务不提供保护，允许任何人查看信息。LDAP为客户机提供了一种身份验证机制，或向目录服务器证明其身份，从而为保护服务器包含的信息的富访问控制铺平了道路。LDAP还支持数据安全(完整性和机密性)服务。

### 1.3 什么时候使用LDAP?

这是一个非常好的问题。通常，当需要通过基于标准的方法集中管理、存储和访问数据时，应该使用目录服务器。

一些常见的例子在整个行业中发现，但不限于:

* 机验证 Machine Authentication
* 用户身份验证 User Authentication
* 用户/系统组 User/System Groups
* 地址本 Address book
* 组织表示 Organization Representation
* 资产跟踪 Asset Tracking
* 电话信息存储 Telephony Information Store
* 用户资源管理 User resource management
* 电子邮件地址查找 E-mail address lookups
* 应用程序配置存储 Application Configuration store
* 交换机配置存储 PBX Configuration store
* 等等...

有各种基于标准的分布式模式文件，但是您总是可以创建自己的模式规范。

总有一些新的方法可以使用目录并应用LDAP原则来解决某些问题，因此这个问题没有简单的答案。

如果有疑问，请在http://www.umich.edu/~dirsvcs/ LDAP /mailinglist.html加入一般LDAP论坛，以获得与LDAP相关的非商业讨论和信息并提问

### 1.4 什么时候不应该使用LDAP?

当您开始发现您需要弯曲目录来执行您的要求时，可能需要重新设计目录。或者如果您只需要一个应用程序来使用和操作您的数据(关于LDAP vs RDBMS的讨论，请阅读LDAP vs RDBMS部分)。

当LDAP是这项工作的合适工具时，这一点就变得很明显了。

### 1.5 LDAP是如何工作的?

LDAP使用客户机-服务器模型。一个或多个LDAP服务器包含组成目录信息树(DIT)的数据。客户端连接到服务器并询问它一个问题。服务器用一个答案和/或一个指向客户端可以获得额外信息的指针(通常是另一个LDAP服务器)进行响应。不管客户机连接到哪个LDAP服务器，它看到的目录视图都是相同的;提供给一个LDAP服务器的名称引用与在另一个LDAP服务器上引用相同的条目。这是全局目录服务的一个重要特性。

### 1.6 X.500呢?

从技术上讲，LDAP是X.500目录服务(OSI目录服务)的目录访问协议。最初，LDAP客户机访问X.500目录服务的网关。这个网关在客户端和网关之间运行LDAP，在网关和X.500服务器之间运行X.500的目录访问协议(DAP)。DAP是一个重量级协议，它在一个完整的OSI协议栈上操作，并且需要大量的计算资源。LDAP被设计为在TCP/IP上操作，并以低得多的成本提供DAP的大部分功能。

虽然LDAP仍然用于通过网关访问X.500目录服务，但LDAP现在更常见地直接在X.500服务器中实现。

独立LDAP守护进程或slapd(8)可以看作是轻量级的X.500目录服务器。也就是说，它不实现X.500的DAP，也不支持完整的X.500模型。

如果您已经在运行X.500 DAP服务，并且希望继续这样做，那么您可以停止阅读本指南。本指南是关于通过slapd(8)运行LDAP而不运行X.500 DAP的。如果您没有运行X.500 DAP，想要停止运行X.500 DAP，或者没有立即运行X.500 DAP的计划，请继续阅读。

可以将数据从LDAP目录服务器复制到X.500 DAP DSA。这需要一个LDAP/DAP网关。OpenLDAP软件不包括这样的网关。

### 1.7 LDAPv2和LDAPv3之间有什么区别?

LDAPv3是在1990年代后期开发的，以替代LDAPv2。LDAPv3向LDAP添加了以下特性:

* 强大的身份验证和数据安全服务通过SASL Strong authentication and data security services via SASL
* 通过TLS (SSL)提供证书认证和数据保安服务 Certificate authentication and data security services via TLS (SSL)
* 通过使用Unicode实现国际化 Internationalization through the use of Unicode
* 推荐和延续 Referrals and Continuations
* 模式发现 Schema Discovery
* 可扩展性(控件、扩展操作等) Extensibility (controls, extended operations, and more)

LDAPv2是历史性的(RFC3494)。由于大多数所谓的LDAPv2实现(包括slapd(8))不符合LDAPv2技术规范，声称支持LDAPv2的实现之间的互操作性受到限制。由于LDAPv2与LDAPv3有很大的不同，同时部署LDAPv2和LDAPv3非常成问题。应该避免使用LDAPv2。LDAPv2在默认情况下是禁用的。

### 1.8 LDAP和RDBMS

这个问题以不同的形式被提出过很多次。然而，最常见的问题是:为什么OpenLDAP不使用关系数据库管理系统(RDBMS)，而使用像LMDB这样的嵌入式键/值存储?一般来说，期望由商业级RDBMS实现的复杂算法能使OpenLDAP更快或更好，同时允许与其他应用程序共享数据。

简单的答案是，使用嵌入式数据库和自定义索引系统使OpenLDAP能够在不丧失可靠性的情况下提供更好的性能和可伸缩性。OpenLDAP使用LMDB并发/事务数据库软件。

现在来看一个更长的答案。我们一直都面临着选择rdbms还是目录的问题。这是一个艰难的选择，没有简单的答案存在。

人们很容易认为目录有一个RDBMS后端可以解决所有问题。然而，它是一头猪。这是因为数据模型非常不同。用关系数据库表示目录数据将需要将数据拆分为多个表。

请考虑一下person对象类。它的定义需要属性类型objectclass、sn和cn，并允许属性类型userPassword、telephoneNumber、seeAlso和description。所有这些属性都是多值的，因此规范化需要将每个属性类型放在单独的表中。

现在您必须为这些表决定适当的键。主键可能是DN的一个组合，但是在大多数数据库实现中，这变得相当低效。

现在的大问题是，从一个条目访问数据需要在不同的磁盘区域上查找。在某些应用程序中，这可能没问题，但在许多应用程序中，性能会受到影响。

唯一可以放在主表条目中的属性类型是那些强制的和单值的。您还可以添加可选的单值属性，如果不存在，则将其设置为NULL或其他值。

但是，等等，条目可以有多个object类，并且它们被组织在一个继承层次结构中。objectclass organizationalPerson的一个条目现在具有person的属性，另外还有一些其他属性和一些以前可选的属性类型现在是强制的。

要做什么吗?我们应该为不同的对象类使用不同的表吗?这样，person将在person表上有一个条目，在organizationalPerson上有另一个条目，等等。或者我们应该把人都放在第二张桌子上?

但是我们如何处理过滤器(cn=*)，其中cn是出现在很多很多对象类中的属性类型。我们应该搜索所有可能的表来匹配条目吗?不是很有吸引力。

一旦达到这一点，就会想到三种方法。一种方法是进行完全的规范化，以便每个属性类型，无论它是什么，都有自己的单独的表。将DN作为主键的一部分的简单方法是极其浪费的，并且需要一种方法，其中条目具有惟一的数字id，用于键和将DNs映射到id的主表。无论如何，当请求来自一个或多个条目的多个属性类型时，这种方法效率非常低。这样的数据库虽然繁琐，但可以通过SQL应用程序进行管理。

第二种方法是将整个条目作为一个blob放在由所有条目共享的表中，而不管对象类是什么，并使用充当第一个表索引的其他表。索引表不是数据库索引，而是完全由LDAP服务器端实现管理的。但是，数据库会因为SQL而无法使用。因此，一个成熟的数据库系统几乎没有任何优势。不需要数据库的全部通用性。最好使用一些轻巧快速的工具，比如LMDB。

另一种完全不同的方法是放弃实现目录数据模型的希望。在这种情况下，LDAP被用作对数据的访问协议，它只提供了表面的目录数据模型。例如，它可能是只读的，或者在允许更新的地方应用限制，例如使单值属性类型允许多个值。或无法向现有条目添加新的对象类或删除其中一个对象类。限制范围从允许的限制(可能是其他地方的访问控制的结果)到完全违反数据模型。但是，它可以是一种方法，提供对其他应用程序使用的现有数据的LDAP访问。但在理解上，我们并没有真正的“目录”。

现有的使用关系数据库的商业LDAP服务器实现要么属于第一种，要么属于第三种。我不知道有哪个实现使用关系数据库来做BDB高效做的事情。对于那些对“第三种方式”(将RDBMS中的现有数据暴露为LDAP树，与经典的LDAP模型相比有一些限制，但使LDAP和SQL应用程序之间的互操作成为可能)感兴趣的人:

OpenLDAP包括后端sql—使之成为可能的后端。它使用ODBC +其他元信息，将LDAP查询转换为RDBMS模式中的SQL查询，提供不同级别的访问--根据您使用的RDBMS和您的模式，从只读访问到完全访问。

有关概念和限制的更多信息，请参见 slapd-sql(5)手册页或后端部分。在back-sql/rdbms_depend/*子目录中还有几个针对几个rdbms的示例。

### 1.9 什么是slapd，它能做什么?

slapd(8)是运行在许多不同平台上的LDAP目录服务器。您可以使用它来提供您自己的目录服务。您的目录可以包含您想要放入的几乎所有内容。您可以将其连接到全局LDAP目录服务，或者自己运行服务。slapd的一些更有趣的特性和功能包括:

* LDAPv3: slapd实现了版本3的轻量级目录访问协议。slapd支持IPv4、IPv6和Unix IPC上的LDAP。

* 简单身份验证和安全层:slapd通过使用SASL支持强身份验证和数据安全(完整性和机密性)服务。slapd的SASL实现利用Cyrus SASL软件，该软件支持许多机制，包括DIGEST-MD5、外部和GSSAPI。

* 传输层安全性:slapd通过使用TLS(或SSL)支持基于证书的身份验证和数据安全性(完整性和机密性)服务。slapd的TLS实现可以利用OpenSSL、GnuTLS或MozNSS软件。

* 拓扑控制:可以根据网络拓扑信息配置slapd来限制套接字层的访问。该特性利用TCP包装器。

* 访问控制:slapd提供了丰富而强大的访问控制功能，允许您控制对数据库中信息的访问。您可以根据LDAP授权信息、IP地址、域名和其他标准控制对条目的访问。slapd同时支持静态和动态访问控制信息。

* 国际化:slapd支持Unicode和语言标记。

* 数据库后端的选择:slapd提供了多种不同的数据库后端供您选择。它们包括MDB，一个分层的高性能事务性数据库后端;高性能的事务数据库后端(已弃用);HDB，层次化的高性能事务后端(已弃用);SHELL，一个后端接口到任意SHELL脚本;PASSWD是PASSWD(5)文件的一个简单后端接口。MDB后端使用LMDB，它是Oracle公司Berkeley DB的高性能替代品。BDB和HDB后端使用Oracle Corporation Berkeley DB。由于LMDB提供了高得多的读写吞吐量和数据可靠性，所以这些后端已经不推荐使用了。

* 多个数据库实例:可以将slapd配置为同时为多个数据库服务。这意味着单个slapd服务器可以使用相同或不同的数据库后端响应LDAP树的许多逻辑上不同部分的请求。

* 通用模块API:如果您需要更多的定制，slapd可以让您轻松地编写自己的模块。slapd由两个不同的部分组成:处理与LDAP客户端的协议通信的前端;以及处理特定任务(如数据库操作)的模块。因为这两部分通过定义良好的C API进行通信，所以您可以编写自己的定制模块，以多种方式扩展slapd。此外，还提供了若干可编程数据库模块。它们允许您使用流行的编程语言(Perl、shell和SQL)向slapd公开外部数据源。

* 线程:slapd是多线程的高性能。一个多线程的slapd过程使用一个线程池处理所有传入的请求。这减少了在提供高性能时所需要的系统开销。

* 复制:可以将slapd配置为维护目录信息的影子副本。这种单提供者/多使用者复制模式在大容量环境中非常重要，在这种环境中，单个slapd安装无法提供必要的可用性或可靠性。对于不能接受单点故障的苛刻环境，还可以使用多提供者复制。对于多提供程序复制，两个或多个节点可以接受写操作，从而允许提供程序级别的冗余。

  slapd包括对基于LDAP同步的复制的支持。

* 代理缓存:slapd可以配置为缓存LDAP代理服务。

* 配置:slapd是高度可配置的，通过一个配置文件，它允许您更改几乎所有您想要更改的。配置选项有合理的默认值，使您的工作更容易。还可以使用LDAP本身动态执行配置，这极大地提高了可管理性。

## 2. 一个快速启动指南

以下是OpenLDAP软件2.4的快速入门指南，包括独立LDAP守护进程slapd(8)。

它将指导您完成安装和配置OpenLDAP软件所需的基本步骤。它应该与本文档的其他章节、手册页和发行版提供的其他资料(例如安装文档)或OpenLDAP网站(http://www.OpenLDAP.org)一起使用，特别是OpenLDAP软件FAQ (http://www.OpenLDAP.org/faq/?file=2)。

如果您打算认真地运行OpenLDAP软件，那么在尝试安装软件之前，应该检查所有文档。

> 注意:本快速入门指南不使用强身份验证，也不使用任何完整性或机密保护服务。这些服务在OpenLDAP管理员指南的其他章节中有描述。

1. 得到软件

   您可以通过遵循OpenLDAP软件下载页面(http://www.openldap.org/software/download/)上的说明来获得该软件的副本。建议新用户从最新版本开始。

2. 解压分布

   选择一个源文件所在的目录，将目录更改到那里，然后使用以下命令解压缩发行版:

   ```
   gunzip -c openldap-VERSION.tgz | tar xvfB -
   ```

   然后重新定位自己到分布目录:

   ```
   cd openldap-VERSION
   ```

   您必须用发行版的版本名替换VERSION。

3. 审查文档

   现在，您应该查看该发行版提供的版权、许可、自述文件和安装文档。版权和许可证提供了关于OpenLDAP软件的可接受使用、复制和担保限制的信息。

   您还应该查看本文档的其他章节。特别是，本文档的构建和安装OpenLDAP软件章节提供了关于必备软件和安装过程的详细信息。

4. 运行配置

   您将需要运行提供的configure脚本来配置在您的系统上构建的发行版。configure脚本接受许多命令行选项，可以启用或禁用可选的软件特性。通常默认值是可以的，但是您可能想要更改它们。要获得configure接受的选项的完整列表，请使用--help选项:

   ```
   ./configure --help
   ```

   然而，鉴于你正在使用这个指南，我们假设你足够勇敢，让配置决定什么是最好的:

   ```
   ./configure
   ```

   假设configure不讨厌您的系统，您可以继续构建软件。如果configure确实抱怨了，那么您可能需要访问软件FAQ安装部分(http://www.openldap.org/faq/?file=8)和/或实际阅读本文档的构建和安装OpenLDAP软件章节。

5. 构建软件。

   下一步是构建软件。这一步有两个部分，首先我们构建依赖关系，然后我们编译软件:

   ```
   make depend
   make
   ```

   两个make命令执行都应没有错误。

6. 测试的构建。

   为了确保构建正确，您应该运行测试套件(只需要几分钟):

   ```
   make test
   ```

   应用于您的配置的测试将会运行，并且应该会通过。有些测试，例如复制测试，可以跳过。

7. 安装软件。

   现在可以安装软件了;这通常需要超级用户特权:

   ```
   su root -c 'make install'
   ```

   现在应该在/usr/local(或配置使用的任何安装前缀)下安装所有组件。

8. 编辑配置文件。

   使用您最喜欢的编辑器编辑所提供的slapd。ldif示例(通常安装为/usr/local/etc/openldap/ slapd.ldif)，以包含MDB数据库定义的表单:

   ```
   dn: olcDatabase=mdb,cn=config
   objectClass: olcDatabaseConfig
   objectClass: olcMdbConfig
   olcDatabase: mdb
   OlcDbMaxSize: 1073741824
   olcSuffix: dc=<MY-DOMAIN>,dc=<COM>
   olcRootDN: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
   olcRootPW: secret
   olcDbDirectory: /usr/local/var/openldap-data
   olcDbIndex: objectClass eq
   ```

   一定要替换 <MY-DOMAIN> 和<COM> 使用适当的域组件或自己的域名. 例：对example.com，使用：

   ```
   dn: olcDatabase=mdb,cn=config
   objectClass: olcDatabaseConfig
   objectClass: olcMdbConfig
   olcDatabase: mdb
   OlcDbMaxSize: 1073741824
   olcSuffix: dc=example,dc=com
   olcRootDN: cn=Manager,dc=example,dc=com
   olcRootPW: secret
   olcDbDirectory: /usr/local/var/openldap-data
   olcDbIndex: objectClass eq
   ```

   如果你的域名包含额外的组件，如eng.uni.edu.eu，请使用:

   ```
   dn: olcDatabase=mdb,cn=config
   objectClass: olcDatabaseConfig
   objectClass: olcMdbConfig
   olcDatabase: mdb
   OlcDbMaxSize: 1073741824
   olcSuffix: dc=eng,dc=uni,dc=edu,dc=eu
   olcRootDN: cn=Manager,dc=eng,dc=uni,dc=edu,dc=eu
   olcRootPW: secret
   olcDbDirectory: /usr/local/var/openldap-data
   olcDbIndex: objectClass eq
   ```

   关于配置slapd(8)的详细信息可以在本文档的配置slapd(5)手册页和配置slapd一章中找到。注意，指定的olcDbDirectory必须在启动slapd(8)之前存在。

9. 导入配置数据库

   现在，通过运行以下命令，可以导入configration数据库供slapd(8)使用:

   ```
   su root -c /usr/local/sbin/slapadd -n 0 -F /usr/local/etc/slapd.d -l /usr/local/etc/openldap/slapd.ldif
   ```

10. SLAPD开始。

    现在可以运行以下命令启动独立的LDAP守护进程slapd(8):

    ```
    su root -c /usr/local/libexec/slapd -F /usr/local/etc/slapd.d
    ```

    要检查服务器是否正在运行并正确配置，可以使用ldapsearch(1)对其运行搜索。默认情况下，ldapsearch安装为/usr/local/bin/ldapsearch:

    ```
    ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
    ```

    注意，命令参数周围使用单引号，以防止shell解释特殊字符。这应该返回:

    ```
    dn:
    namingContexts: dc=example,dc=com
    ```

    关于运行slapd(8)的详细信息可以在本文档的slapd(8)手册页和运行slapd一章中找到。

11. 将初始项添加到目录中。

    可以使用ldapadd(1)将条目添加到LDAP目录。ldapadd期望以LDIF形式输入。我们将分两步来完成:

    1. 创建一个LDIF文件

    2. 运行 Ldapadd

    使用您最喜欢的编辑器，并创建一个LDIF文件，其中包含:

    ```
    dn: dc=<MY-DOMAIN>,dc=<COM>
    objectclass: dcObject
    objectclass: organization
    o: <MY ORGANIZATION>
    dc: <MY-DOMAIN>
    
    dn: cn=Manager,dc=<MY-DOMAIN>,dc=<COM>
    objectclass: organizationalRole
    cn: Manager
    ```

    替换<MY-DOMAIN>、<COM>和<MY ORGANIZATION>，如：

    ```
    dn: dc=example,dc=com
    objectclass: dcObject
    objectclass: organization
    o: Example Company
    dc: example
    
    dn: cn=Manager,dc=example,dc=com
    objectclass: organizationalRole
    cn: Manager
    ```

    现在，可以运行ldapadd(1)将这些条目插入到目录中。

    ```
    ldapadd -x -D "cn=Manager,dc=<MY-DOMAIN>,dc=<COM>" -W -f example.ldif
    ```

    替换其中的<MY-DOMAIN>和<COM>，系统将提示您输入在slap.conf中指定的“secret”。如：

    ```
    ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f example.ldif
    ```

    系统将提示您输入在 slapd.conf中指定的“secret”。

    其中 example.ldif 是您上面创建的文件。

    关于目录创建的其他信息可以在本文档的数据库创建和维护工具一章中找到。

12. 看看它是否有效。

    现在，我们已经准备好验证添加的条目是否在目录中。您可以使用任何LDAP客户机来完成这项工作，但是我们的示例使用了ldapsearch(1)工具。记住将dc=example,dc=com替换为正确的站点值:

    ```
    ldapsearch -x -b 'dc=example,dc=com' '(objectclass=*)'
    ```

    这个命令将搜索和检索数据库中的每个条目。



现在可以使用ldapadd(1)或另一个LDAP客户机添加更多条目，试验各种配置选项、后端安排等。

注意，默认情况下，slapd(8)数据库授予除超级用户(由rootdn配置指令指定)之外的所有用户读访问权。强烈建议您建立控制来限制对授权用户的访问。访问控制在访问控制一章中讨论。还建议您阅读安全注意事项、使用SASL和使用TLS部分。

下面的章节提供了关于制作、安装和运行slapd(8)的更详细的信息。

## 3. 大图-配置选择

本节简要介绍各种LDAP目录配置，以及独立LDAP守护进程slapd(8)如何与其他环境相适应。

### 3.1 本地目录服务

在此配置中，您运行一个slapd(8)实例，它仅为您的本地域提供目录服务。它不以任何方式与其他目录服务器交互。此配置如图3.1所示。

```
图3.1 <Figure 3.1 - Local service configuration.png>
```

如果您刚刚开始使用这个配置(快速启动指南为您提供了这个配置)，或者如果您想提供本地服务，但对连接到世界的其他地方不感兴趣，那么可以使用这个配置。如果您愿意，稍后可以很容易地升级到另一个配置。

### 3.2 具有引用的本地目录服务

在此配置中，您运行一个slapd(8)实例，它为您的本地域提供目录服务，并将其配置为返回对其他能够处理请求的服务器的引用。您可以自己运行此服务或使用提供给您的服务。此配置如图3.2所示。

```
图3.2 <Figure 3.2 - Local service with referrals.png>
```
如果您希望提供本地服务并参与全局目录，或者希望将从属项的责任委托给另一台服务器，请使用此配置。

### 3.3 复制目录服务

slapd(8)包括对基于LDAP同步的复制(称为syncrepl)的支持，该复制可用于在多个目录服务器上维护目录信息的影子副本。在其最基本的配置中，提供者是一个syncrepl提供者，一个或多个使用者(或影子)是syncrepl使用者。图3.3显示了提供者-使用者配置的示例。还支持多提供者配置。

```
图3.3 <Figure 3.2 - Local service with referrals.png>
```

在单个slapd(8)实例不能提供所需的可靠性或可用性的情况下，可以将此配置与前两种配置结合使用。

### 3.4 分布式本地目录服务

在此配置中，本地服务被划分为更小的服务，每个服务都可以被复制，并通过上级和下级引用粘在一起。

## 4. 构建和安装OpenLDAP软件

本章详细介绍了如何构建和安装OpenLDAP软件包，包括独立的LDAP守护进程slapd(8)。构建和安装OpenLDAP软件需要几个步骤:安装必备软件、配置OpenLDAP软件本身、制作并最终安装。下面的部分详细描述了这个过程。

### 4.1 软件的获取和提取

您可以从项目的下载页面http://www.openldap.org/software/download/获取OpenLDAP软件，或者直接从项目的FTP服务获取OpenLDAP软件，网址是ftp://ftp.openldap.org/pub/OpenLDAP/。

该项目提供了两组通用的包。该项目在新特性和bug修复可用时发布版本。尽管项目采取了一些措施来提高这些版本的稳定性，但是通常只有在发布之后才会出现问题。稳定版本是最新的版本，它通过通用的使用证明了稳定性。

OpenLDAP软件的用户可以选择安装最合适的系列，这取决于他们对最新特性和演示的稳定性的需求。

下载完OpenLDAP软件后，您需要从压缩的归档文件中提取发行版，并将您的工作目录更改为发行版的顶目录:

```
gunzip -c openldap-VERSION.tgz | tar xf -
cd openldap-VERSION
```

您必须用发行版的版本名替换VERSION。

现在，您应该查看该发行版提供的版权、许可、自述文件和安装文档。版权和许可证提供了关于OpenLDAP软件的可接受使用、复制和担保限制的信息。自述文件和安装文档提供了必备软件和安装过程的详细信息。

### 4.2 必备软件

OpenLDAP软件依赖于第三方发布的许多软件包。根据您打算使用的特性，您可能必须下载和安装许多附加软件包。本节详细介绍了通常需要安装的第三方软件包。但是，对于最新的先决条件信息，应该参考自述文件。请注意，其中一些第三方软件包可能依赖于其他软件包。按照附带的安装说明安装每个软件包。

#### 4.2.1 准备传输层安全

OpenLDAP客户端和服务器需要安装OpenSSL、GnuTLS或MozNSS TLS库来提供传输层安全服务。尽管一些操作系统可能将这些库作为基本系统的一部分或可选的软件组件提供，但OpenSSL、GnuTLS和Mozilla NSS通常需要单独安装。

OpenSSL可以从http://www.openssl.org/获得。GnuTLS可以从http://www.gnu.org/software/gnutls/获得。Mozilla NSS可以从http://developer.mozilla.org/en/NSS获得。

除非OpenLDAP的配置检测到一个可用的TLS库，否则OpenLDAP软件将不完全符合LDAPv3。

#### 4.2.2 简单的身份验证和安全层

OpenLDAP客户机和服务器需要安装Cyrus SASL库来提供简单的身份验证和安全层服务。尽管一些操作系统可能将此库作为基本系统的一部分或可选的软件组件提供，Cyrus SASL通常需要单独安装。

Cyrus SASL可从http://asg.web.cmu.edu/sasl/sasl-library.html获得。Cyrus SASL将使用预先安装的OpenSSL和Kerberos/GSSAPI库。

除非OpenLDAP的配置检测到一个可用的Cyrus SASL安装，否则OpenLDAP软件将不完全符合LDAPv3。

#### 4.2.3 Kerberos身份验证服务

OpenLDAP客户机和服务器支持Kerberos身份验证服务。特别是，OpenLDAP支持Kerberos V GSS-API SASL身份验证机制，即GSSAPI机制。除了Cyrus SASL库之外，该特性还需要Heimdal或MIT Kerberos V库。

Heimdal Kerberos可从http://www.pdc.kth.se/heimdal/获得。可以从http://web.mit.edu/kerberos/www/获得MIT Kerberos。

强烈建议使用强身份验证服务，比如Kerberos提供的服务。

#### 4.2.4 数据库软件

OpenLDAP的slapd(8) MDB主数据库后端使用OpenLDAP源中包含的LMDB软件。不需要下载任何附加软件来获得MDB支持。

OpenLDAP的slapd(8) BDB和HDB不赞成使用的数据库后端需要Oracle公司的Berkeley DB。如果在配置时不可用，您将无法使用这些已弃用的数据库后端构建slapd(8)。

您的操作系统可能在基本系统中提供一个受支持的Berkeley DB版本，或者作为一个可选的软件组件。如果没有，您必须自己获取并安装它。如果需要的话，可以从Oracle公司的Berkeley DB下载页面下载。

Oracle公司提供了几个版本。Berkeley DB version 6.0.20及以后版本使用的软件许可证与LDAP技术不兼容，不应该与OpenLDAP一起使用。

> 注意:有关更多信息，请参阅推荐的OpenLDAP软件依赖版本。

#### 4.2.5 线程

OpenLDAP被设计用来利用线程。OpenLDAP支持POSIX pthreads、Mach CThreads和许多其他种类的线程。如果configure找不到合适的线程子系统，它会抱怨。如果出现这种情况，请咨询OpenLDAP FAQ http://www.openldap.org/faq/中的软件|安装|平台提示部分。

#### 4.2.6 TCP包装器

如果预先安装，slapd(8)支持TCP包装器(IP级别访问控制过滤器)。建议对包含非公共信息的服务器使用TCP包装器或其他ip级访问过滤器(如ip级防火墙提供的过滤器)。

### 4.3 运行配置

现在，您可能应该使用--help选项运行配置脚本。这将为您提供一个选项列表，您可以在构建OpenLDAP时更改这些选项。可以使用这种方法启用或禁用OpenLDAP的许多特性。

```
./configure --help
```

configure脚本还在命令行和环境中查找某些变量。这些包括:

| **Variable** | **Description**                   |
| ------------ | --------------------------------- |
| CC           | Specify alternative C Compiler    |
| CFLAGS       | Specify additional compiler flags |
| CPPFLAGS     | Specify C Preprocessor flags      |
| LDFLAGS      | Specify linker flags              |
| LIBS         | Specify additional libraries      |

现在使用所需的配置选项或变量运行configure脚本。

```
./configure [options] [variable=value ...]
```

作为示例，让我们假设希望安装具有BDB后端和TCP包装器支持的OpenLDAP。默认情况下，BDB是启用的，而TCP包装器没有启用。因此，我们只需要指定--enablewrappers来包含TCP Wrappers支持:

```
./configure --enable-wrappers
```

但是，这将无法找到未安装在系统目录中的相关软件。例如，如果TCP包装器头和库分别安装在/usr/local/include和/usr/local/lib中，配置脚本通常应该调用如下:

```
./configure --enable-wrappers \
                CPPFLAGS="-I/usr/local/include" \
                LDFLAGS="-L/usr/local/lib -Wl,-rpath,/usr/local/lib"
```

配置脚本通常会自动检测适当的设置。如果在这个阶段有问题，请参考任何特定于平台的提示并检查配置选项(如果有的话)。

### 4.4 构建软件

一旦你运行了配置脚本，输出的最后一行应该是:

```
Please "make depend" to build dependencies
```

如果最后一行输出不匹配，则configure失败，您将需要检查其输出以确定出错的地方。在配置成功完成之前，不应该继续操作。

要建立依赖关系，运行:

```
make depend
```

现在构建软件，这一步将实际编译OpenLDAP。

```
make
```

您应该仔细检查此命令的输出，以确保所有内容都正确构建。注意，这个命令构建LDAP库和关联的客户机以及slapd(8)。

### 4.5 测试软件

一旦正确地配置并成功地创建了软件，您就应该运行测试套件来验证构建。

```
make test
```

应用于您的配置的测试将会运行，并且应该会通过。如果您的配置不支持某些测试(如复制测试)，则可以跳过它们。

### 4.6 安装软件

一旦成功地测试了软件，就可以安装它了。您将需要对运行configure时指定的安装目录具有写权限。默认情况下，OpenLDAP软件安装在/usr/local中。如果您使用--prefix配置选项更改了该设置，它将安装在您提供的位置。

通常，安装需要超级用户特权。在顶层的OpenLDAP源目录中，键入:

```
su root -c 'make install'
```

并在需要时输入适当的密码。

您应该仔细检查此命令的输出，以确保所有内容都正确安装。默认情况下，您将在/usr/local/etc/openldap中找到slapd(8)的配置文件。有关更多信息，请参阅配置slapd一章。

## 5. slapd配置

一旦构建并安装了软件，您就可以配置slapd(8)以便在您的站点上使用了。

OpenLDAP 2.3及以后版本已经过渡到使用动态运行时配置引擎 slapd-config(5)。slapd-config (5)

- 完全启用ldap
- 是否使用标准LDAP操作进行管理
- 将配置数据存储在LDIF数据库中，通常存储在' /usr/local/etc/openldap/slapd.d '目录。
- 允许动态更改所有slapd的配置选项，通常不需要重新启动服务器就可以使更改生效。

本章介绍了 slapd-config(5)配置系统的一般格式，然后详细介绍了常用的设置。

旧样式的slapd.conf(5)文件仍然受到支持，但是不赞成使用它，并且在未来的OpenLDAP版本中将取消对它的支持。通过slapd.conf(5)配置slapd(8)将在下一章进行描述。

关于如何让slapd自动从slapd.conf(5)转换到slapd-config(5)的信息，请参考slapd(8)。

> 注意:尽管slapd-config(5)系统将其配置存储为(基于文本的)LDIF文件，但您不应该直接编辑任何LDIF文件。配置更改应该通过LDAP操作来执行，例如ldapadd(1)、ldapdelete(1)或ldapmodify(1)。

> 注意:如果您的OpenLDAP安装需要使用一个或多个未更新的后端或覆盖来使用slapd-config(5)系统，那么您将需要继续使用旧的slapd.conf(5)配置系统。从OpenLDAP 2.4.33开始，所有官方后端都已更新。可能还有其他贡献的或还没有更新的实验覆盖。

### 5.1 配置布局

slapd配置存储为具有预定义模式和DIT的特殊LDAP目录。有一些特定的对象类用于携带全局配置选项、模式定义、后端和数据库定义以及其他组合项。配置树示例如图5.1所示。

```
图5.1 <Figure 5.1 - Sample configuration tree.png>
```

其他对象可能是配置的一部分，但是为了清晰起见，在插图中省略了。

slapd-config配置树有一个非常特定的结构。树的根目录名为cn=config，包含全局配置设置。附加设置包含在单独的子条目中:

- 动态加载的模块

  ​	只有在使用--enable-modules选项配置软件时，才可以使用这些。

- 模式定义

  ​	cn=schema,cn=config条目包含系统模式(在slapd中硬编码的所有模式)。

  ​	cn=schema、cn=config的子条目包含从配置文件加载或在运行时添加的用户架构。

- 后端特定配置

- 特定于数据库的配置

  ​	覆盖在数据库条目的子条目中定义。

  ​	数据库和覆盖层还可能有其他杂项子元素。

LDIF文件的通常规则适用于配置信息:忽略以'#'字符开头的注释行。如果一行以一个空格开始，它被认为是前一行的延续(即使前一行是注释)，并且单前导空格被删除。条目由空行分隔。

配置LDIF的总体布局如下:

```
# global configuration settings
dn: cn=config
objectClass: olcGlobal
cn: config
<global config settings>

# schema definitions
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema
<system schema>

dn: cn={X}core,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: {X}core
<core schema>

# additional user-specified schema
...

# backend definitions
dn: olcBackend=<typeA>,cn=config
objectClass: olcBackendConfig
olcBackend: <typeA>
<backend-specific settings>

# database definitions
dn: olcDatabase={X}<typeA>,cn=config
objectClass: olcDatabaseConfig
olcDatabase: {X}<typeA>
<database-specific settings>

# subsequent definitions and settings
...
```

上面列出的一些条目的名称中有一个数字索引“{X}”。虽然大多数配置设置都具有内在的顺序依赖关系(即，在设置后续设置之前，一个设置必须生效)，但LDAP数据库本质上是无序的。数值索引用于在配置数据库中执行一致的排序，以便保留所有的排序依赖项。在大多数情况下，并不需要提供索引;它将根据条目创建的顺序自动生成。

配置指令被指定为单个属性的值。slapd配置中使用的大多数属性和对象类的名称中都有一个前缀“olc”(OpenLDAP配置)。通常，属性和旧式的slapd.conf配置关键字之间存在一一对应关系，使用关键字作为属性名称，并附加“olc”前缀。

配置指令可以带参数。如果是，则参数之间用空格分隔。如果参数包含空格，参数应该像这样用双引号括起来。在接下来的描述中，括号中显示了应该被实际文本替换的参数。

该发行版包含一个示例配置文件，该文件将安装在/usr/local/etc/openldap目录中。在/usr/local/etc/openldap/schema目录中还提供了许多包含模式定义(属性类型和对象类)的文件。

### 5.2 配置指令

本节详细介绍常用的配置指令。要获得完整的列表，请参阅slapd-config(5)手册页。本节将以自顶向下的顺序处理配置指令，从cn=config条目中的全局指令开始。每个指令将被描述，连同它的默认值(如果有)和它的使用示例。

#### 5.2.1 cn=config

此条目中包含的指令通常应用于整个服务器。它们中的大多数是面向系统或连接的，而不是与数据库相关的。此条目必须具有olcGlobal对象类。

##### 5.2.1.1 olcIdleTimeout: \<integer\>

指定在强制关闭空闲客户端连接之前等待的秒数。默认值0禁用此功能。

##### 5.2.1.2 olcLogLevel: \<level\>

这个指令指定调试语句和操作统计信息应该被syslod(目前记录到syslogd(8) LOG_LOCAL4工具)的级别。您必须配置了OpenLDAP--enable-debug(缺省值)才能工作(除了始终启用的两个统计级别之外)。日志级别可以指定为整数或关键字。可以使用多个日志级别，并且这些级别是附加的。要显示什么级别对应什么类型的调试，用-d调用slapd ?或者参考下表。&lt;level\>的可能值是:

| **Level** | **Keyword**    | **Description**              |
| --------- | -------------- | ---------------------------- |
| -1        | any            | 启用全部调试                 |
| 0         |                | 没有调试                     |
| 1         | (0x1 trace)    | 跟踪函数调用                 |
| 2         | (0x2 packets)  | 调试包处理                   |
| 4         | (0x4 args)     | 重量级的跟踪调试             |
| 8         | (0x8 conns)    | 连接管理                     |
| 16        | (0x10 BER)     | 打印发送和接收的数据包       |
| 32        | (0x20 filter)  | 搜索过滤器处理               |
| 64        | (0x40 config)  | 配置处理                     |
| 128       | (0x80 ACL)     | 访问控制列表处理             |
| 256       | (0x100 stats)  | 数据日志连接/操作/结果       |
| 512       | (0x200 stats2) | 发送的统计信息日志条目       |
| 1024      | (0x400 shell)  | 打印与shell后端通信          |
| 2048      | (0x800 parse)  | 打印条目解析调试             |
| 16384     | (0x4000 sync)  | syncrepl消费者处理           |
| 32768     | (0x8000 none)  | 只记录任何日志级别设置的消息 |

所需的日志级别可以作为单个整数输入，该整数将所需的级别(十进制或十六进制)组合为一个整数列表(在内部使用)，或者作为方括号之间显示的名称列表，例如

```
olcLogLevel 129
olcLogLevel 0x81
olcLogLevel 128 1
olcLogLevel 0x80 0x1
olcLogLevel acl trace
```

是等价的。

例子:

```
olcLogLevel -1
```

这将导致记录大量调试信息。

```
olcLogLevel conns filter
```

只要记录连接和搜索过滤器处理。

```
olcLogLevel none
```

不管配置的日志级别是什么，都要记录这些日志消息。这与在不发生日志记录时将日志级别设置为0不同。至少需要None级别来记录高优先级消息。

默认值:

```
olcLogLevel stats
```

默认情况下配置了基本状态日志记录。但是，如果没有定义olcLogLevel，就不会发生日志记录(相当于0级)。

##### 5.2.1.3 olcReferral \<URI\>

这个指令指定当slapd找不到处理请求的本地数据库时要传回的引用。

例子:

```
olcReferral: ldap://root.openldap.org
```

这将把非本地查询引用到OpenLDAP项目中的全局根LDAP服务器。智能LDAP客户机可以在该服务器上重新询问其查询，但请注意，大多数客户机只知道如何处理简单的LDAP url，其中包含主机部分和可选的专有名称部分。

##### 5.2.1.4 示例条目

```
dn: cn=config
objectClass: olcGlobal
cn: config
olcIdleTimeout: 30
olcLogLevel: Stats
olcReferral: ldap://root.openldap.org
```

#### 5.2.2 cn=module

如果在配置slapd时启用了对动态加载模块的支持，则可以使用cn =module条目来指定要加载的模块集。模块条目必须具有olcModuleList对象类。

##### 5.2.2.1 olcModuleLoad: \<filename\>

指定要加载的可动态加载模块的名称。文件名可以是绝对路径名，也可以是简单文件名。在olcModulePath指令指定的目录中搜索非绝对名称。

##### 5.2.2.2 olcModulePath: \<pathspec\>

指定要搜索可加载模块的目录列表。通常路径是用冒号分隔的，但这取决于操作系统。

##### 5.2.2.3 示例条目

```
dn: cn=module{0},cn=config
objectClass: olcModuleList
cn: module{0}
olcModuleLoad: /usr/local/lib/smbk5pwd.la

dn: cn=module{1},cn=config
objectClass: olcModuleList
cn: module{1}
olcModulePath: /usr/local/lib:/usr/local/lib/slapd
olcModuleLoad: accesslog.la
olcModuleLoad: pcache.la
```

#### 5.2.3 cn=schema

schema条目包含在slapd中硬编码的所有模式定义。因此，该条目中的值是由slapd生成的，因此不需要在配置文件中提供模式值。不过，仍然必须定义条目，以作为要添加到其下的用户定义模式的基础。模式条目必须具有olcSchemaConfig对象类。

##### 5.2.3.1 olcAttributeTypes: \<[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Attribute Type Description\>

这个指令定义了一个属性类型。有关如何使用此指令的信息，请参阅模式规范一章。

##### 5.2.3.2 olcObjectClasses: \<[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Object Class Description\>

这个指令定义了一个对象类。有关如何使用此指令的信息，请参阅模式规范一章。

##### 5.2.3.3 示例条目

```
dn: cn=schema,cn=config
objectClass: olcSchemaConfig
cn: schema

dn: cn=test,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: test
olcAttributeTypes: ( 1.1.1
  NAME 'testAttr'
  EQUALITY integerMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.27 )
olcAttributeTypes: ( 1.1.2 NAME 'testTwo' EQUALITY caseIgnoreMatch
  SUBSTR caseIgnoreSubstringsMatch SYNTAX 1.3.6.1.4.1.1466.115.121.1.44 )
olcObjectClasses: ( 1.1.3 NAME 'testObject'
  MAY ( testAttr $ testTwo ) AUXILIARY )
```



#### 5.2.4 后端特定指令

后端指令应用于相同类型的所有数据库实例，根据指令的不同，可能会被数据库指令覆盖。后端条目必须具有olcBackendConfig对象类。

##### 5.2.4.1 olcBackend: \<type\>

这个指令命名一个特定于后端的配置条目。\<type\>应该是表5.2中列出的受支持的后端类型之一。

| **类型** | **描述**                      |
| -------- | ----------------------------- |
| bdb      | Berkeley DB事务后端(已弃用)   |
| config   | Slapd配置后端                 |
| dnssrv   | DNS SRV后端                   |
| hdb      | bdb后端的层次结构变体(已弃用) |
| ldap     | 轻量级目录访问协议(代理)后端  |
| ldif     | 轻量级数据交换格式后端        |
| mdb      | 内存映射数据库后端            |
| meta     | 元目录后端                    |
| monitor  | 监控端                        |
| passwd   | 提供对passwd的只读访问(5)     |
| perl     | Perl编程的后端                |
| shell    | Shell(extern程序)后端         |
| sql      | SQL编程后端                   |

例子：

```
olcBackend: bdb
```

此条目没有定义其他指令。特定的后端类型可以为其特定用途定义额外的属性，但到目前为止还没有定义任何属性。因此，这些指令通常不会出现在任何实际配置中。

##### 5.2.4.2 示例条目

```
dn: olcBackend=bdb,cn=config
objectClass: olcBackendConfig
olcBackend: bdb
```

#### 5.2.5 特定于数据库的指令

本节中的指令由每种类型的数据库支持。数据库条目必须具有olcDatabaseConfig对象类。

##### 5.2.5.1 olcDatabase: [{\<index>}]\<type>

这个指令命名一个特定的数据库实例。可以提供数值{\<index\>}来区分相同类型的多个数据库。通常索引可以省略，slapd会自动生成索引。\<type>应该是表5.2中列出的支持后端类型之一或前端类型。

前端是一个特殊的数据库，用于保存应该应用于所有其他数据库的数据库级选项。后续数据库定义还可能覆盖一些前端设置。

配置数据库也很特殊;即使没有显式配置，配置数据库和前端数据库都是隐式创建的，并且在任何其他数据库之前创建。

例子:

```
olcDatabase: bdb
```

这标志着一个新的BDB数据库实例的开始。

##### 5.2.5.2 olcAccess: to \<what> [ by \<who> [\<accesslevel>] [\<control>] ]+

这个指令授予一个或多个请求者(由\<who\>指定)对一组条目和/或属性的访问权(由\<what>指定)。有关基本用法，请参阅本指南的访问控制部分。

> 注意:如果没有指定olcAccess指令，默认的访问控制策略“to * by * read”允许所有用户(通过身份验证的和匿名的)读访问。

> 注意:前端定义的访问控制附加到所有其他数据库的控制。

##### 5.2.5.3 olcReadonly { TRUE | FALSE }

该指令将数据库置于“只读”模式。任何修改数据库的尝试都将返回一个“不愿意执行”的错误。如果在使用者上设置，syncrepl发送的修改仍然会发生。

默认值:

```
olcReadonly: FALSE
```

##### 5.2.5.4 olcRootDN: \<DN>

此指令指定不受此数据库操作的访问控制或管理限制限制的DN。DN不需要引用数据库中的条目，甚至也不需要引用目录中的条目。DN可以引用一个SASL标识。

Entry-based例子:

```
olcRootDN: cn=Manager,dc=example,dc=com
```

SASL-based例子：

```
olcRootDN: uid=root,cn=example.com,cn=digest-md5,cn=auth
```

有关SASL身份验证身份的信息，请参阅SASL身份验证（SASL Authentication section）一节。

##### 5.2.5.5 olcRootPW: \<password>

此指令可用于为rootdn指定DN的密码(当rootdn被设置为数据库中的DN时)。

例子:

```
olcRootPW: secret
```

还可以提供 [RFC2307](http://www.rfc-editor.org/rfc/rfc2307.txt) 格式的密码散列。可以使用slappasswd(8)生成密码散列。

例子：

```
olcRootPW: {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```

该散列是使用如下命令生成的：

```
slappasswd -s secret
```

##### 5.2.5.6 olcSizeLimit: \<integer>

此指令指定从搜索操作返回的最大项数。

默认值:

```
olcSizeLimit: 500
```

更多细节请参见本指南的限制部分(Limit section)和slapd-config(5)。

##### 5.2.5.7 olcSuffix: \<dn suffix>

此指令指定将传递到此后端数据库的查询的DN后缀。可以给出多个后缀行，通常每个数据库定义至少需要一个后缀行。(一些后端类型，如frontend和monitor使用硬编码的后缀，在配置中可能不会被覆盖。)

例子:

```
olcSuffix: dc=example,dc=com
```

DN以“dc=example,dc=com”结尾的查询将被传递到此后端。

> 注意:当要传递查询的后端被选中时，slapd会按照配置的顺序查看每个数据库定义中的后缀值。因此，如果一个数据库后缀是另一个的前缀，那么在配置中它必须出现在该后缀的后面。

##### 5.2.5.8 olcSyncrepl

```
olcSyncrepl: rid=<replica ID>
                provider=ldap[s]://<hostname>[:port]
                [type=refreshOnly|refreshAndPersist]
                [interval=dd:hh:mm:ss]
                [retry=[<retry interval> <# of retries>]+]
                searchbase=<base DN>
                [filter=<filter str>]
                [scope=sub|one|base]
                [attrs=<attr list>]
                [attrsonly]
                [sizelimit=<limit>]
                [timelimit=<limit>]
                [schemachecking=on|off]
                [bindmethod=simple|sasl]
                [binddn=<DN>]
                [saslmech=<mech>]
                [authcid=<identity>]
                [authzid=<identity>]
                [credentials=<passwd>]
                [realm=<realm>]
                [secprops=<properties>]
                [starttls=yes|critical]
                [tls_cert=<file>]
                [tls_key=<file>]
                [tls_cacert=<file>]
                [tls_cacertdir=<path>]
                [tls_reqcert=never|allow|try|demand]
                [tls_cipher_suite=<ciphers>]
                [tls_crlcheck=none|peer|all]
                [logbase=<base DN>]
                [logfilter=<filter str>]
                [syncdata=default|accesslog|changelog]
```

此指令通过将当前slapd(8)建立为运行syncrepl复制引擎的复制使用者站点，将当前数据库指定为提供者内容的使用者。提供程序数据库位于提供程序参数指定的提供程序站点。使用LDAP内容同步协议使使用者数据库与提供者内容保持最新。有关协议的更多信息，请参见[RFC4533](http://www.rfc-editor.org/rfc/rfc4533.txt) 。

rid参数用于标识复制消费者服务器内的当前syncrepl指令，其中副本ID\>唯一标识由当前syncrepl指令描述的syncrepl规范。\<ID\>复制品;非负的，长度不超过三位小数。

提供程序参数指定包含提供程序内容的复制提供程序站点作为LDAP URI。provider参数指定了一个方案、一个主机和一个端口，提供者slapd实例可以在这些端口中找到。主机名可以使用域名或IP地址。例如ldap://provider.example.com:389或ldaps://192.168.1.1:636。如果\<port\>，则使用标准LDAP端口号(389或636)。请注意，syncrepl使用使用者发起的协议，因此其规范位于使用者之上。

syncrepl使用者的内容使用搜索规范作为其结果集定义。使用者slapd将根据搜索规范向提供者slapd发送搜索请求。搜索规范包括searchbase、scope、filter、attrs、attrsonly、sizelimit和timelimit参数，和正常的搜索规范一样。searchbase参数没有默认值，必须始终指定。scope默认为sub, filter默认为(objectclass=*)， attrs默认为“*，+”，以复制所有用户和操作属性，attrsonly默认不设置。sizelimit和timelimit都默认为“unlimited”，只能指定正整数或“unlimited”。

LDAP内容同步协议有两种操作类型:refreshOnly和refreshAndPersist。操作类型由类型参数指定。在刷新操作中，在每次同步操作完成后，将按间隔时间定期重新调度下一个同步搜索操作。interval由interval参数指定。默认设置为一天。在刷新和持久化操作中，同步搜索在提供者slapd实例中保持持久。对提供者的进一步更新将向使用者slapd生成searchResultEntry，作为对持久同步搜索的搜索响应。

如果在复制过程中出现错误，用户将根据重试参数(重试间隔列表)尝试重新连接。和检索对。例如，retry="60 10 300 3"允许用户在前10次中每60秒重试一次，然后在停止重试之前每300秒重试三次。+ in &lt;#检索\>意味着不确定的重试次数，直到成功。

通过打开schemacheckin参数，可以在LDAP同步使用者站点强制执行模式检查。如果它是打开的，那么每个复制条目都将被检查其模式，因为该条目存储在消费者上。使用者中的每个条目都应该包含模式定义所需的那些属性。如果它被关闭，条目将在不检查模式一致性的情况下被存储。默认设置为关闭。

binddn参数为syncrepl搜索提供程序slapd提供了要绑定的DN。它应该是一个对提供者数据库中的复制内容具有读访问权的DN。

bindmethod是simple还是sasl，这取决于在连接到提供者slapd实例时使用的是简单的基于密码的身份验证还是sasl身份验证。

除非有足够的数据完整性和机密性保护(例如TLS或IPsec)，否则不应该使用简单的身份验证。简单的身份验证需要binddn和凭证参数的规范。

一般建议使用SASL身份验证。SASL身份验证需要使用saslmech参数指定一种机制。根据机制的不同，可以分别使用authcid和凭证指定身份验证标识和/或凭证。authzid参数可用于指定授权标识。

realm参数指定某个机制对其中的标识进行身份验证的领域。secprops参数指定Cyrus SASL安全属性。

starttls参数指定使用starttls扩展操作在对提供者进行身份验证之前建立TLS会话。如果提供了关键参数，如果StartTLS请求失败，会话将中止。否则，syncrepl会话将继续，而不需要TLS。tls_reqcert设置默认为“demand”，其他TLS设置默认为与slapd主TLS设置相同。

使用者可以查询数据修改的日志，而不是复制整个条目。这种操作模式称为delta syncrepl。除了上述参数外，还必须针对将要使用的日志适当地设置logbase和logfilter参数。如果日志符合slapo-accesslog(5)日志格式，则必须将syncdata参数设置为“accesslog”，如果日志符合过时的changelog格式，则必须设置为“changelog”。如果syncdata参数被忽略或设置为“默认”，那么日志参数将被忽略。

bdb、hdb和mdb后端支持syncrepl复制机制。

有关如何使用此指令的更多信息，请参阅本指南的LDAP同步复制一章（ [LDAP Sync Replication](https://www.openldap.org/doc/admin24/guide.html#LDAP Sync Replication) ）。

##### 5.2.5.9 olcTimeLimit: \<integer>

这个指令指定了slapd响应搜索请求所花费的最大秒数(实时)。如果在这个时间内请求没有完成，将返回一个指示超过时间限制的结果。

默认值:

```
olcTimeLimit: 3600
```

更多细节请参见本指南的限制部分和slapd-config(5)（[Limits](https://www.openldap.org/doc/admin24/guide.html#Limits) section）。

##### 5.2.5.10 olcUpdateref: \<URL>

这个指令只适用于一个副本(或影子)slapd(8)实例。它指定了返回给客户端的URL，客户端在副本上提交更新请求。如果指定多次，则提供每个URL。

例子:

```
olcUpdateref:   ldap://provider.example.net
```

##### 5.2.5.11 示例条目

```
dn: olcDatabase=frontend,cn=config
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: frontend
olcReadOnly: FALSE

dn: olcDatabase=config,cn=config
objectClass: olcDatabaseConfig
olcDatabase: config
olcRootDN: cn=Manager,dc=example,dc=com
```

#### 5.2.6 BDB和HDB数据库指令

此类指令同时适用于BDB和HDB数据库。除了上面定义的通用数据库指令外，它们还用于olcDatabase条目。有关BDB/HDB配置指令的完整参考，请参阅slapd-bdb(5)。除了olcDatabaseConfig对象类之外，BDB和HDB数据库条目还必须分别具有olcBdbConfig和olcHdbConfig对象类。

##### 5.2.6.1 olcDbDirectory: \<directory>

这个指令指定包含数据库和相关索引的BDB文件所在的目录。

默认值:

```
olcDbDirectory: /usr/local/var/openldap-data
```

##### 5.2.6.2 olcDbCachesize: \<integer>

此指令指定BDB后端数据库实例维护的内存缓存条目的大小。

默认值:

```
olcDbCachesize: 1000
```

##### 5.2.6.3 olcDbCheckpoint: \<kbyte> \<min>

这个指令指定检查BDB事务日志的频率。检查点操作将数据库缓冲区刷新到磁盘，并在日志中写入检查点记录。如果有任何一种情况发生，就会出现检查点。数据已写入或已写入。从最后一个检查点到现在已经过去几分钟了。这两个参数默认为零，在这种情况下它们将被忽略。当\<min参数为非零时，内部任务将运行执行检查点所需的时间。有关更多细节，请参阅Berkeley DB参考指南。

例子:

```
olcDbCheckpoint: 1024 10
```

##### 5.2.6.4 olcDbConfig: \<DB_CONFIG setting>

这个属性指定了一个要放在数据库目录的DB_CONFIG文件中的配置指令。在服务器启动时，如果还不存在这样的文件，就会创建DB_CONFIG文件，并将此属性中的设置写入其中。如果文件存在，它的内容将被读取并显示在这个属性中。属性是多值的，以适应多个配置指令。这里没有提供缺省值，但是必须使用适当的设置来获得最佳服务器性能。

对该属性所做的任何更改都将写入DB_CONFIG文件，并导致数据库环境被重置，以便更改能够立即生效。如果环境缓存很大，并且最近还没有被检查点，这个重置操作可能会花费很长时间。在使用LDAP修改来更改此属性之前，使用Berkeley DB db_checkpoint实用程序手工执行单个检查点可能是可取的。

例子:

```
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097512
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
```

在本例中，BDB缓存设置为10MB, BDB事务日志缓冲区大小设置为2MB，事务日志文件存储在/var/tmp/bdb-log目录中。还设置了一个标志，告诉BDB在事务日志文件的内容被检查点并且不再需要它们时立即删除它们。如果不进行此设置，事务日志文件将继续累积，直到其他清理过程将其删除为止。有关db_archive命令的详细信息，请参阅Berkeley DB文档。有关Berkeley DB标志的完整列表，请参见http://www.oracle.com/technology/documentation/berkeley-db/db/api_c/env_set_flags.html

理想情况下，BDB缓存必须至少与数据库的工作集一样大，日志缓冲区的大小应该足够大，能够容纳大多数事务而不会溢位，并且日志目录必须位于与主数据库文件分开的物理磁盘上。数据库目录和日志目录都应该与用于常规系统活动(如根、引导或交换文件系统)的磁盘分开。有关更多细节，请参阅FAQ-o-Matic和Berkeley DB文档。

##### 5.2.6.5 olcDbNosync: { TRUE | FALSE }

此选项导致磁盘上的数据库内容不会在更改时立即与内存中的更改同步。将此选项设置为TRUE可以以牺牲数据完整性为代价提高性能。此指令与下面配置具有相同的效果

```
olcDbConfig: set_flags DB_TXN_NOSYNC
```

##### 5.2.6.6 olcDbIDLcacheSize: \<integer>

在索引槽中指定内存中索引缓存的大小。默认值是零。较大的值将加速索引条目的频繁搜索。最佳大小取决于数据库的数据和搜索特性，但是使用条目缓存大小三倍的数字是一个很好的起点。

例子:

```
olcDbIDLcacheSize: 3000
```

##### 5.2.6.7 olcDbIndex: {\<attrlist> | default} [pres,eq,approx,sub,none]

此指令指定要为给定属性维护的索引。如果仅有一个\<attrlist>，则维护默认索引。索引关键字对应于LDAP搜索筛选器中可能使用的常见匹配类型。

例子:

```
olcDbIndex: default pres,eq
olcDbIndex: uid
olcDbIndex: cn,sn pres,eq,sub
olcDbIndex: objectClass eq
```

此指令指定要为给定属性维护的索引。第一行设置要维护的缺省索引集，以表示和相等。第三行导致为cn和sn属性类型维护present、相等和子字符串索引。第四行生成objectClass属性类型的相等索引。

不等式匹配没有索引关键字。通常这些匹配不使用索引。但是，有些属性确实支持基于相等索引为不等匹配建立索引。

子字符串索引可以更显式地指定为subinitial、subany或subfinal，对应于子字符串匹配过滤器的三个可能组件。子初始化索引只索引出现在属性值开头的子字符串。subfinal索引只索引出现在属性值末尾的子字符串，而subany索引出现在值的任何位置的子字符串。

注意，默认情况下，为属性设置索引也会影响该属性的每个子类型。例如，在name属性上设置相等索引会导致cn、sn和其他所有从name继承的属性都被索引。

默认情况下，不维护索引。一般建议在objectClass上至少维护一个相等索引。

```
olcDbindex: objectClass eq
```

此指令指定要为给定属性维护的索引。应该配置与数据库上使用的最常见搜索对应的其他索引。大多数应用程序不使用在线状态搜索，所以在线状态索引通常不是很有用。

如果在运行slapd时更改了该设置，则将运行一个内部任务来生成更改后的索引数据。在索引器执行其工作时，所有服务器操作可以正常继续。如果在索引任务完成之前停止了slapd，则必须使用slapindex工具手动完成索引。

##### 5.2.6.8 olcDbLinearIndex: { TRUE | FALSE }

此指令指定要为给定属性维护的索引。如果该设置为真，slapindex将一次索引一个属性。当启用时，每个索引属性都将通过多次遍历整个数据库来单独处理。当数据库大小超过BDB缓存大小时，此选项可以改进slapindex性能。当BDB缓存足够大时，不需要这个选项，并且会降低性能。另外，在默认情况下，slapadd执行完整的索引，因此不需要单独运行slapindex。使用此选项，slapadd不做索引，必须使用slapindex。

##### 5.2.6.9 olcDbMode: { \<octal> | \<symbolic> }

此指令指定要为给定属性维护的索引。此指令指定新创建的数据库索引文件应具有的文件保护模式。这可以是0600或-rw-的形式

默认值:

```
olcDbMode: 0600
```

##### 5.2.6.10 olcDbSearchStack: \<integer>

此指令指定要为给定属性维护的索引。指定用于搜索筛选器计算的堆栈深度。搜索过滤器在堆栈上计算以适应嵌套的和/或子句。为每个服务器线程分配一个单独的堆栈。堆栈的深度决定在不需要任何额外内存分配的情况下评估过滤器的复杂程度。嵌套深度超过搜索堆栈深度的过滤器将导致为特定搜索操作分配一个单独的堆栈。这些独立的分配可能会对服务器性能产生重大的负面影响，但是指定太多的堆栈也会消耗大量内存。在32位计算机上，每次搜索每级使用512K字节，在64位计算机上每级使用1024K字节。默认的堆栈深度是16，因此32位和64位机器上每个线程分别使用8MB或16MB。此外，单个堆栈槽的512KB大小是由编译时常量设置的，如果需要，该常量可以更改;要使更改生效，必须重新编译代码。

默认值:

```
 olcDbSearchStack: 16
```

##### 5.2.6.11 olcDbShmKey: \<integer>

为共享内存BDB环境指定一个密钥。默认情况下，BDB环境使用内存映射文件。如果指定了一个非零值，它将被用作键来标识将容纳该环境的共享内存区域。

例子:

```
olcDbShmKey: 42
```

##### 5.2.6.12 示例条目

```
dn: olcDatabase=hdb,cn=config
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: hdb
olcSuffix: dc=example,dc=com
olcDbDirectory: /usr/local/var/openldap-data
olcDbCacheSize: 1000
olcDbCheckpoint: 1024 10
olcDbConfig: set_cachesize 0 10485760 0
olcDbConfig: set_lg_bsize 2097152
olcDbConfig: set_lg_dir /var/tmp/bdb-log
olcDbConfig: set_flags DB_LOG_AUTOREMOVE
olcDbIDLcacheSize: 3000
olcDbIndex: objectClass eq
```

### 5.3 配置示例

下面是一个配置示例，其中穿插了解释性文本。它定义了两个数据库来处理X.500树的不同部分;两者都是BDB数据库实例。所显示的行号仅供参考，并不包含在实际文件中。首先，全局配置部分:

```
1.    # example config file - global configuration entry
2.    dn: cn=config
3.    objectClass: olcGlobal
4.    cn: config
5.    olcReferral: ldap://root.openldap.org
6.
```

第1行是一个注释。第2-4行将其标识为全局配置条目。第5行中的olcReferral:指令意味着，对以下定义的数据库的非本地查询将被引用到主机root.openldap.org上标准端口(389)上运行的LDAP服务器。第6行是空白行，表示该条目的结束。

```
7.    # internal schema
8.    dn: cn=schema,cn=config
9.    objectClass: olcSchemaConfig
10.    cn: schema
11.
```

第7行是一个注释。第8-10行将其标识为模式子树的根。此条目中的实际模式定义被硬编码到slapd中，因此这里不指定其他属性。第11行是空白行，表示该条目的结束。

```
12.    # include the core schema
13.    include: file:///usr/local/etc/openldap/schema/core.ldif
14.
```

第12行是一个注释。第13行是一个LDIF include指令，它以LDIF格式访问核心模式定义。第14行是空行。

接下来是数据库定义。第一个数据库是特殊的前端数据库，其设置全局应用于所有其他数据库。

```
15.    # global database parameters
16.    dn: olcDatabase=frontend,cn=config
17.    objectClass: olcDatabaseConfig
18.    olcDatabase: frontend
19.    olcAccess: to * by * read
20.
```

第15行是一个注释。第16-18行将该条目标识为全局数据库条目。第19行是一个全局访问控制。它适用于所有条目(在任何适用的特定于数据库的访问控制之后)。第20行是空行。

下一个条目定义了配置后端。

```
21.    # set a rootpw for the config database so we can bind.
22.    # deny access to everyone else.
23.    dn: olcDatabase=config,cn=config
24.    objectClass: olcDatabaseConfig
25.    olcDatabase: config
26.    olcRootPW: {SSHA}XKYnrjvGT3wZFQrDD5040US592LxsdLy
27.    olcAccess: to * by * none
28.
```

第21-22行是注释。第23-25行将该条目标识为配置数据库条目。第26行定义了这个数据库的超级用户密码。(DN默认为“cn=config”。)第27行拒绝对该数据库的所有访问，因此只有超级用户才能访问该数据库。这已经是配置数据库的默认访问权限了。这里列出它只是为了说明，并重申，除非显式地配置了验证超级用户的方法，否则config数据库将无法访问。)

第28行是空行。

下一个条目定义了一个BDB后端，它将处理对树“dc=example,dc=com”部分内容的查询。索引要维护几个属性，userPassword属性要防止未经授权的访问。

```
29.    # BDB definition for example.com
30.    dn: olcDatabase=bdb,cn=config
31.    objectClass: olcDatabaseConfig
32.    objectClass: olcBdbConfig
33.    olcDatabase: bdb
34.    olcSuffix: dc=example,dc=com
35.    olcDbDirectory: /usr/local/var/openldap-data
36.    olcRootDN: cn=Manager,dc=example,dc=com
37.    olcRootPW: secret
38.    olcDbIndex: uid pres,eq
39.    olcDbIndex: cn,sn pres,eq,approx,sub
40.    olcDbIndex: objectClass eq
41.    olcAccess: to attrs=userPassword
42.      by self write
43.      by anonymous auth
44.      by dn.base="cn=Admin,dc=example,dc=com" write
45.      by * none
46.    olcAccess: to *
47.      by self write
48.      by dn.base="cn=Admin,dc=example,dc=com" write
49.      by * read
50.
```

第29行是一个注释。第30-33行将该条目标识为BDB数据库配置条目。第34行指定要传递到该数据库的查询的DN后缀。第35行指定数据库文件所在的目录。

第36和37行标识数据库超级用户条目和相关密码。此条目不受访问控制、大小或时间限制的限制。

第38行到第40行表示各种属性要维护的索引。

第41行到第49行为这个数据库中的条目指定了访问控制。对于所有适用的条目，userPassword属性可由条目本身和“admin”条目写入。它可以用于身份验证/授权目的，但否则不可读。所有其他属性都可以由条目和“admin”条目写入，但是可以由所有用户(通过身份验证或未通过身份验证)读取。

第50行是空白行，表示该条目的结束。

下一个条目定义了另一个BDB数据库。该数据库处理涉及dc=example、dc=net子树的查询，但由与第一个数据库相同的实体管理。注意，如果没有第60行，那么由于第19行的全局访问规则，读访问将被允许。

```
51.    # BDB definition for example.net
52.    dn: olcDatabase=bdb,cn=config
53.    objectClass: olcDatabaseConfig
54.    objectClass: olcBdbConfig
55.    olcDatabase: bdb
56.    olcSuffix: dc=example,dc=net
57.    olcDbDirectory: /usr/local/var/openldap-data-net
58.    olcRootDN: cn=Manager,dc=example,dc=com
59.    olcDbIndex: objectClass eq
60.    olcAccess: to * by users read
```

### 5.4 将旧样式的 slapd.conf(5)文件转换为cn=config格式

在转换为cn=config格式之前，您应该确保配置后端在现有的配置文件中已正确配置。虽然配置后端始终存在于slapd中，但默认情况下它只能由其rootDN访问，并且没有分配缺省凭据，因此除非您显式地配置对其进行身份验证的方法，否则它将不可用。

如果还没有数据库配置部分，可以在slap .conf的末尾添加类似这样的内容

```
 database config
 rootpw VerySecret
```

> 注意:由于配置后端可以用于将任意代码加载到slapd进程中，因此非常重要的是要小心地保护用于访问它的任何凭据。由于简单的密码容易受到密码猜测攻击，因此通常最好忽略rootpw，而只对配置rootDN使用SASL身份验证。

可以使用slaptest(8)或任何一个slap工具将现有的slapd.conf(5)文件转换为新的格式:

```
slaptest -f /usr/local/etc/openldap/slapd.conf -F /usr/local/etc/openldap/slapd.d
```

测试您可以访问cn=config下的条目，使用默认的rootdn和上面配置的rootpw:

```
ldapsearch -x -D cn=config -w VerySecret -b cn=config
```

然后可以丢弃旧的slapd.conf(5)文件。如果不使用默认目录路径，请确保使用-F选项启动slapd(8)以指定配置目录。

> 注意:从slapd.conf格式转换为slapd格式时。d格式，任何包含的文件也将被集成到最终的配置数据库中。

## 6. slapd配置文件

本章描述了通过slapd.conf(5)配置文件配置slapd(8)。conf(5)已被弃用，只有当您的站点需要一个尚未更新的后端来与更新的slapd-config(5)系统一起工作时，才应该使用。通过slapd-config(5)配置slapd(8)在前一章中有描述。

conf(5)文件通常安装在/usr/local/etc/openldap目录中。可以通过slapd(8)的命令行选项指定另一个配置文件位置。

### 6.1 配置文件格式

slapd.conf(5)文件由三种类型的配置信息组成:全局的、特定于后端的和特定于数据库的。首先指定全局信息，然后是与特定后端类型关联的信息，然后是与特定数据库实例关联的信息。全局指令可以在后台和/或数据库指令中被覆盖，后台指令可以被数据库指令覆盖。

空白行和以“#”字符开头的注释行将被忽略。如果一行以空格开始，则认为它是前一行的延续(即使前一行是注释)。

slapd.conf的一般格式如下:

```
# global configuration directives
<global config directives>

# backend definition
backend <typeA>
<backend-specific directives>

# first database definition & config directives
database <typeA>
<database-specific directives>

# second database definition & config directives
database <typeB>
<database-specific directives>

# second database definition & config directives
database <typeA>
<database-specific directives>

# subsequent backend & database definitions & config directives
...
```

配置指令可以带参数。如果是，则用空格分隔。如果参数包含空格，参数应该像这样用双引号括起来。如果参数包含双引号或反斜杠字符' \'，字符前应该有反斜杠字符' \'。

该发行版包含一个示例配置文件，该文件将安装在/usr/local/etc/openldap目录中。在/usr/local/etc/openldap/schema目录中还提供了许多包含模式定义(属性类型和对象类)的文件。

### 6.2 配置文件的指示

本节详细介绍常用的配置指令。要获得完整的列表，请参阅slapd.conf(5)手册页。本节将配置文件指令分为全局、特定于后端和特定于数据的类别，描述每个指令及其默认值(如果有的话)，并给出一个使用示例。

#### 6.2.1 全局指令

本节描述的指令适用于所有后端和数据库，除非在后端或数据库定义中被特别重写。应该被实际文本替换的参数显示在括号中\<>。

##### 6.2.1.1. access to \<what> [ by \<who> [\<accesslevel>] [\<control>] ]+

这个指令授予一个或多个请求者(由\<who>指定)对一组条目和/或属性的访问权(由\<what>指定)。有关基本用法，请参阅本指南的访问控制部分（[Access Control](https://www.openldap.org/doc/admin24/guide.html#Access Control) section）。

> 注意:如果没有指定访问指令，默认的访问控制策略* by * read允许所有身份验证用户和匿名用户读访问。

##### 6.2.1.2 attributetype <[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Attribute Type Description>

这个指令定义了一个属性类型。有关如何使用此指令的信息，请参阅模式规范一章（[Schema Specification](https://www.openldap.org/doc/admin24/guide.html#Schema Specification) chapter）。

##### 6.2.1.3 idletimeout \<integer>

指定在强制关闭空闲客户端连接之前等待的秒数。默认值为0的idletimeout禁用此特性。

##### 6.2.1.4 include \<filename>

这个指令指定slapd在继续执行当前文件的下一行之前，应该从给定文件中读取额外的配置信息。所包含的文件应该遵循正常的slapd配置文件格式。该文件通常用于包含包含模式规范的文件。

> 注意:在使用这个指令时，你应该小心——嵌套的include指令的数量没有很小的限制，并且没有做循环检测。

##### 6.2.1.5 loglevel \<level>

这个指令指定调试语句和操作统计信息应该被syslod(目前记录到syslogd(8) LOG_LOCAL4工具)的级别。您必须配置了OpenLDAP--enable-debug(缺省值)才能工作(除了始终启用的两个统计级别之外)。日志级别可以指定为整数或关键字。可以使用多个日志级别，并且这些级别是附加的。要显示哪些数字对应于哪种调试，请使用-d调用slapd ?或者参考下表。整数的可能值是:



| **Level** | **Keyword**    | **Description**              |
| --------- | -------------- | ---------------------------- |
| -1        | any            | 启用全部调试                 |
| 0         |                | 没有调试                     |
| 1         | (0x1 trace)    | 跟踪函数调用                 |
| 2         | (0x2 packets)  | 调试包处理                   |
| 4         | (0x4 args)     | 重量级的跟踪调试             |
| 8         | (0x8 conns)    | 连接管理                     |
| 16        | (0x10 BER)     | 打印发送和接收的数据包       |
| 32        | (0x20 filter)  | 搜索过滤器处理               |
| 64        | (0x40 config)  | 配置处理                     |
| 128       | (0x80 ACL)     | 访问控制列表处理             |
| 256       | (0x100 stats)  | 数据日志连接/操作/结果       |
| 512       | (0x200 stats2) | 发送的统计信息日志条目       |
| 1024      | (0x400 shell)  | 打印与shell后端通信          |
| 2048      | (0x800 parse)  | 打印条目解析调试             |
| 16384     | (0x4000 sync)  | syncrepl消费者处理           |
| 32768     | (0x8000 none)  | 只记录任何日志级别设置的消息 |

所需的日志级别可以作为单个整数输入，该整数将所需的级别(十进制或十六进制)组合为一个整数列表(在内部使用)，或者作为方括号之间显示的名称列表，例如

```
loglevel 129
loglevel 0x81
loglevel 128 1
loglevel 0x80 0x1
loglevel acl trace
```

是等价的。

例子:

```
loglevel -1
```

这将导致记录大量调试信息。

```
loglevel conns filter
```

只要记录连接和搜索过滤器处理。

```
loglevel none
```

不管配置的日志级别是什么，都要记录这些日志消息。这与在不发生日志记录时将日志级别设置为0不同。至少需要None级别来记录高优先级消息。

默认值:

```
loglevel stats
```

默认情况下配置了基本状态日志记录。但是，如果没有定义日志级别，就不会发生日志记录(相当于0级别)。

##### 6.2.1.6 objectclass <[RFC4512](http://www.rfc-editor.org/rfc/rfc4512.txt) Object Class Description>

这个指令定义了一个对象类。有关如何使用此指令的信息，请参阅模式规范一章（[Schema Specification](https://www.openldap.org/doc/admin24/guide.html#Schema Specification) chapter ）。

##### 6.2.1.7 referral \<URI>

这个指令指定当slapd找不到处理请求的本地数据库时要传回的引用。

例子:

```
referral ldap://root.openldap.org
```

这将把非本地查询引用到OpenLDAP项目中的全局根LDAP服务器。智能LDAP客户机可以在该服务器上重新询问其查询，但请注意，大多数客户机只知道如何处理简单的LDAP url，其中包含主机部分和可选的专有名称部分。

##### 6.2.1.8 sizelimit \<integer>

此指令指定从搜索操作返回的最大项数。

默认值:

```
sizelimit 500
```

参见本指南的限制部分（[Limits](https://www.openldap.org/doc/admin24/guide.html#Limits) section）和slapd.conf(5)了解更多细节。

##### 6.2.1.9 timelimit \<integer>

这个指令指定了slapd响应搜索请求所花费的最大秒数(实时)。如果在这个时间内请求没有完成，将返回一个指示超过时间限制的结果。

默认值:

```
timelimit 3600
```

参见本指南的限制部分（[Limits](https://www.openldap.org/doc/admin24/guide.html#Limits) section）和slapd.conf(5)了解更多细节。

#### 6.2.2 一般后台指令

本节中的指令只适用于它们定义的后端。它们得到后端每种类型的支持。后端指令适用于同一类型的所有数据库实例,而且,根据该指令,可能会被数据库指令覆盖。

##### 6.2.2.1 backend \<type>

该指令标志了后端声明的开始。\<type>应该是表6.2中列出的支持后端类型之一。

| **类型** | **描述**                      |
| -------- | ----------------------------- |
| bdb      | Berkeley DB事务后端(已弃用)   |
| config   | Slapd配置后端                 |
| dnssrv   | DNS SRV后端                   |
| hdb      | bdb后端的层次结构变体(已弃用) |
| ldap     | 轻量级目录访问协议(代理)后端  |
| ldif     | 轻量级数据交换格式后端        |
| mdb      | 内存映射数据库后端            |
| meta     | 元目录后端                    |
| monitor  | 监控端                        |
| passwd   | 提供对passwd的只读访问(5)     |
| perl     | Perl编程的后端                |
| shell    | Shell(extern程序)后端         |
| sql      | SQL编程后端                   |

例子：

```
backend mdb
```

这标志着新的MDB后端定义的开始。

#### 6.2.3 通用数据库指令

本节中的指令只适用于它们定义的数据库。它们得到了每一种数据库的支持。

##### 6.2.3.1 database \<type>

该指令标志着数据库实例声明的开始。& gt;应该是表6.2中列出的支持后端类型之一。

例子:

```
database mdb
```

这标志着新的MDB数据库实例声明的开始。

##### 6.2.3.2 limits \<selector> \<limit> [\<limit> [...]]

根据操作的发起者或基本DN指定时间和大小限制。

请参阅本指南的限制部分（[Limits](https://www.openldap.org/doc/admin24/guide.html#Limits) section）和slapd.conf(5)。

##### 6.2.3.3 readonly { on | off }

这一指令将数据库放入“了”模式。任何修改数据库的尝试都将返回“不愿意执行”错误。如果在一个消费者上设置,同步发送的修改仍然会发生。

默认情况:

```
readonly off
```

##### 6.2.3.4 rootdn \<DN>

该指令指定不受访问控制或管理限制对该数据库操作的限制的DN。DN不需要引用该数据库中的条目,也不需要引用目录中的条目。DN可以指SASL标识。

基于入口的例子:

```
rootdn "cn=Manager,dc=example,dc=com"
```

基于SASL的例子：

```
rootdn "uid=root,cn=example.com,cn=digest-md5,cn=auth"
```

参见SASL身份验证部分（[SASL Authentication](https://www.openldap.org/doc/admin24/guide.html#SASL Authentication) section）,了解SASL身份验证身份的信息。

##### 6.2.3.5 rootpw \<password>

该指令可用于为rootdn指定DN的密码(当rootdn被设置为数据库中的DN时)。

例子:

```
rootpw: secret
```

还可以提供 [RFC2307](http://www.rfc-editor.org/rfc/rfc2307.txt) 格式的密码散列。可以使用slappasswd(8)生成密码散列。

例子：

```
rootpw: {SSHA}ZKKuqbEKJfKSXhUbHG3fG8MDn9j1v4QN
```

该散列是使用如下命令生成的：

```
slappasswd -s secret
```

##### 6.2.3.6 suffix \<dn suffix>

该指令指定将传递给这个后端数据库的查询的DN后缀。可以给出多个后缀行,每个数据库定义都需要至少一个。

例子:

```
suffix "dc=example,dc=com"
```

在“dc =示例中,dc = com”的DN查询将传递给这个后端。

> 注意:当后端将查询传递给被选中时,slapd将查看在文件中出现的顺序中的每个数据库定义的后缀线(s)。因此,如果一个数据库后缀是另一个数据库后缀,它必须在配置文件中出现。

##### 6.2.3.7 syncrepl

```
 syncrepl rid=<replica ID>
                provider=ldap[s]://<hostname>[:port]
                searchbase=<base DN>
                [type=refreshOnly|refreshAndPersist]
                [interval=dd:hh:mm:ss]
                [retry=[<retry interval> <# of retries>]+]
                [filter=<filter str>]
                [scope=sub|one|base]
                [attrs=<attr list>]
                [exattrs=<attr list>]
                [attrsonly]
                [sizelimit=<limit>]
                [timelimit=<limit>]
                [schemachecking=on|off]
                [network-timeout=<seconds>]
                [timeout=<seconds>]
                [bindmethod=simple|sasl]
                [binddn=<DN>]
                [saslmech=<mech>]
                [authcid=<identity>]
                [authzid=<identity>]
                [credentials=<passwd>]
                [realm=<realm>]
                [secprops=<properties>]
                [keepalive=<idle>:<probes>:<interval>]
                [starttls=yes|critical]
                [tls_cert=<file>]
                [tls_key=<file>]
                [tls_cacert=<file>]
                [tls_cacertdir=<path>]
                [tls_reqcert=never|allow|try|demand]
                [tls_cipher_suite=<ciphers>]
                [tls_crlcheck=none|peer|all]
                [tls_protocol_min=<major>[.<minor>]]
                [suffixmassage=<real DN>]
                [logbase=<base DN>]
                [logfilter=<filter str>]
                [syncdata=default|accesslog|changelog]
```

该指令通过将当前的slapd(8)建立为一个运行syncrepl复制引擎的复制用户站点,指定当前数据库作为提供者内容的使用者。提供者数据库位于提供者参数指定的复制提供者站点。消费者数据库使用LDAP内容同步协议保持最新的提供者内容。有关协议的更多信息,请参阅RFC4533（[RFC4533](http://www.rfc-editor.org/rfc/rfc4533.txt)）。

rid参数用于识别复制消费者服务器中的当前syncrepl指令;在where &lt;副本ID唯一标识当前syncrepl指令所描述的syncrepl规范。复制ID是非负的,长度不超过3个十进制数字。

提供者参数指定包含提供者内容的复制提供者站点作为LDAP URI。提供者参数指定一个方案、一个主机和一个可以找到提供者slapd实例的端口。域名或IP地址可以用于&lt;hostname。例子是ldap:/ / provider.example.com:389或ldaps:/ / 192.168.1.1:636。如果不给出,则使用标准的LDAP端口号(389或636)。注意,syncrepl使用了消费者启动的协议,因此它的规范位于消费者。

syncrepl消费者的内容是使用搜索规范作为结果集定义的。消费者slapd将根据搜索规范将搜索请求发送给提供者slapd。搜索规范包括searchbase、scope、filter、attrs、attrs、attrsonly、sizelimit和timelimit参数,如正常的搜索规范。searchbase参数没有默认值,必须指定。范围默认为sub,过滤器默认值(objectclass = *),attrs默认值为“* +”,复制所有用户和操作属性,而attrsonly则默认为unset。可指定“无限”的尺寸限制和时间限制默认值,只有正整数或“无限”。exattrs选项也可以用来指定从传入条目中省略的属性。

LDAP内容同步协议有两种操作类型:只刷新和更新。操作类型由类型参数指定。在只刷新的操作中,下一个同步搜索操作在每次同步操作结束后定期重新安排。间隔由间隔参数指定。默认情况下,它将被设定为一天。在更新和持久化操作中,在提供者slapd实例中,同步搜索仍然是持久化的。对提供者的进一步更新将为消费者slapd生成searchResultEntry,作为对持续同步搜索的搜索响应。

如果在复制过程中出现错误,则消费者将尝试根据retry参数进行重新连接,这是一个列表,并重新尝试\<interval>例如,retry =“6010300 3”让消费者在前10次每60秒重试,然后在接下来的三次再试每300秒再试一次。在获得成功的时候,意味着不确定的重试。

通过打开schemachecking参数,可以在LDAP同步消费者站点执行模式检查。如果打开,每个复制的条目都会被检查它的模式,因为条目存储在消费者身上。消费者的每一个条目都应该包含模式定义所要求的属性。如果关闭了,条目将在没有检查模式一致性的情况下存储。默认是关闭。

网络超时参数设置了消费者将等待建立与提供者的网络连接的时间。一旦建立了连接,超时参数决定消费者将等待初始绑定请求完成多长时间。这些参数的默认值来自ldap.conf(5)。

binddn参数提供DN绑定到syncrepl搜索到提供者slapd。它应该是一个DN,它读取了提供者数据库中复制内容的访问。

bindmethod是简单的或sasl,这取决于在连接到提供者slapd实例时是否使用基于简单的基于密码的身份验证或sasl身份验证。

除非有足够的数据完整性和保密保护(例如TLS或IPsec),否则不应该使用简单的身份验证。简单的身份验证需要对binddn和凭据参数进行规范。

一般推荐SASL身份验证。SASL身份验证需要使用saslmech参数的机制规范。根据机制,可以分别使用authcid和凭据指定身份验证身份和/或凭证。可以使用authzid参数来指定授权身份。

域参数指定了一个域,其中一个特定的机制验证了内部的身份。sec道具参数指定Cyrus SASL安全属性。

keepalive参数设置了空闲、探针和间隔的值,用于检查套接字是否还存在;空闲时间是在TCP开始发送keepalive探针之前保持空闲的时间的数量;探针是在断开连接之前发送的最大的保护探针的最大数量;间隔是单个保存探针之间的间隔。只有一些系统支持对这些值的定制;其他的参数被忽略,系统的设置被使用。例如,保持生命=“240:的”将会在240秒的空闲活动后,每30秒发送一探针10次。如果接收到的探测器没有响应,连接将会被删除。

starttls参数指定在对提供者进行身份验证之前,使用starttls扩展操作来建立TLS会话。如果提供了关键参数,如果StartTLS请求失败,会话将被中止。否则syncrepl会话将继续没有TLS。tls_reqcert将默认设置为“需求”,其他TLS设置默认与主slapd TLS设置相同。

后缀参数允许消费者从一个远程目录中提取条目,它的DN后缀不同于本地目录。匹配searchbase的远程条目DNs的部分将被后缀的后缀替换。

消费者可以查询数据修改的日志,而不是复制整个条目。这种操作方式被称为delta syncrepl。除了上面的参数之外,必须适当地为将要使用的日志设置logbase和logfilter参数。如果日志符合slapo-accesslog(5)日志格式或“changelog”,如果日志符合过时的changelog格式,则必须将syncdata参数设置为“accesslog”。如果省略了syncdata参数,或者设置为“默认”,那么日志参数就被忽略了。

syncrepl复制机制由bdb、hdb和mdb支持。

请参阅本指南的LDAP同步复制章（[LDAP Sync Replication](https://www.openldap.org/doc/admin24/guide.html#LDAP Sync Replication) chapter ）,了解更多关于如何使用该指令的信息。

##### 6.2.3.8 updateref \<URL>

该指令只适用于副本(或阴影)slapd(8)实例。它指定返回给客户机的URL,它向副本提交更新请求。如果指定多个次数,则提供每个URL。

例子:

```
updateref       ldap://provider.example.net
```

#### 6.2.4 BDB和HDB数据库指令

这个类别中的指令只适用于BDB和HDB数据库。也就是说,他们必须遵循一个“数据库bdb”或“数据库hdb”行,在任何后续的“后端”或“数据库”行之前出现。对于BDB / HDB配置指令的完整引用,请参见slapd-bdb(5)。

##### 6.2.4.1 directory \<directory>

该指令指定包含数据库和相关索引的BDB文件的目录。

默认情况:

```
directory /usr/local/var/openldap-data
```

### 6.3 配置文件的例子

下面是一个示例配置文件,使用解释性文本进行插入。它定义了两个数据库来处理X.500树的不同部分,两者都是BDB数据库实例。所示的行号仅供参考,不包括在实际文件中。第一,全局配置部分:

```
1.    # example config file - global configuration section
2.    include /usr/local/etc/schema/core.schema
3.    referral ldap://root.openldap.org
4.    access to * by * read
```

第一行是注释。第2行包括另一个包含核心模式定义的配置文件。第3行的转介指令意味着在下面定义的数据库中没有本地查询的查询将被调用在主机root.openldap.org上运行在标准端口(389)的LDAP服务器上。

第4行是一个全局访问控制。它适用于所有条目(在任何适用的数据库特定访问控制之后)。

配置文件的下一部分定义了一个BDB后端,它将处理“dc = com”部分中“dc = com”的内容的查询。数据库将被复制到两个复制的slapds,一个在truelies,另一个在判断日。索引将被维护为多个属性,用户密码属性将受到保护,不受未经授权的访问。

```
5.    # BDB definition for the example.com
6.    database bdb
7.    suffix "dc=example,dc=com"
8.    directory /usr/local/var/openldap-data
9.    rootdn "cn=Manager,dc=example,dc=com"
10.    rootpw secret
11.    # indexed attribute definitions
12.    index uid pres,eq
13.    index cn,sn pres,eq,approx,sub
14.    index objectClass eq
15.    # database access control definitions
16.    access to attrs=userPassword
17.        by self write
18.        by anonymous auth
19.        by dn.base="cn=Admin,dc=example,dc=com" write
20.        by * none
21.    access to *
22.        by self write
23.        by dn.base="cn=Admin,dc=example,dc=com" write
24.        by * read
```

第5行是一个评论。数据库定义的开始由第6行中的数据库关键字标记。第7行指定通过到这个数据库的查询的DN后缀。第8行指定数据库文件将存在的目录。

第9行和第10行识别数据库超级用户输入和相关密码。此条目不受访问控制、大小或时间限制限制。

第12行到14行表示索引以维护各种属性。

第16行到24行指定该数据库中的条目的访问控制。对于所有适用的条目,用户密码属性都是通过条目本身和“admin”条目来编写的。它可以用于身份验证/授权用途,但否则不可读。所有其他属性都是通过条目和“admin”条目编写的,但每个用户都可以读取(通过身份验证)。

示例配置文件的下一部分定义了另一个BDB数据库。这个处理涉及dc =示例的查询,dc = net subtree,但由与第一个数据库相同的实体管理。注意,如果没有第39行,将允许读取访问,因为第4行的全局访问规则。

```
33.    # BDB definition for example.net
34.    database bdb
35.    suffix "dc=example,dc=net"
36.    directory /usr/local/var/openldap-data-net
37.    rootdn "cn=Manager,dc=example,dc=com"
38.    index objectClass eq
39.    access to * by users read
```

## 7. 运行slapd

### 7.1 命令行选项

### 7.2 开始slapd

### 7.3 停止slapd

## 8. 访问控制

### 8.1 介绍

### 8.2 通过静态配置进行访问控制

#### 8.2.1中 控制访问权限

#### 8.2.2 向谁授予访问权限

#### 8.2.3 获得授予的权利

#### 8.2.4 访问控制评估

#### 8.2.5 访问控制的例子

### 8.3 通过动态配置进行访问控制

#### 8.3.1 控制访问权限

#### 8.3.2 向谁授予访问权限

#### 8.3.3 获得授予的权利

#### 8.3.4 访问控制评估

#### 8.3.5 访问控制的例子

#### 8.3.6 访问控制要求

8.4 访问控制常见示例

8.4.1 基本的acl

8.4.2 匹配匿名用户和经过身份验证的用户

8.4.3 控制rootdn访问

8.4.4 管理对组的访问

8.4.5 授予对属性子集的访问权

8.4.6 允许用户写入他们下面的所有条目

8.4.7 允许创建条目

8.4.8 在访问控制中使用正则表达式的提示

8.4.9 基于安全强度因素(ssf)授予和拒绝访问

8.4.10 当事情没有按照预期进行时



8.5 集合--基于关系的授予权限

8.5.1 组的组

8.5.2 不使用DN语法对acl进行分组

8.5.3 以下引用



\9. 限制

9.1 介绍

9.2 软硬限制

9.3 全球范围

9.4 数据库的限制

9.4.1的 指定限制适用于谁

老的 指定时间限制

9.4.3 指定大小限制

9.4.4 大小限制和分页结果



9.5 限制示例配置

9.5.1 简单的全球范围

9.5.2 全球硬和软限制

9.5.3 给特定用户更大的限制

9.5.4 限制谁可以做分页搜索



9.6 进一步的信息



\10. 数据库创建和维护工具

10.1 通过LDAP创建数据库

10.2 离线创建数据库

10.2.1 slapadd程序

10.2.2 slapindex程序

10.2.3 slapcat程序



10.3 LDIF文本输入格式



\11. 后端

11.1 伯克利数据库后端

11.1.1 概述

11.1.2 back-bdb / back-hdb配置

11.1.3 进一步的信息



11.2 LDAP

11.2.1 概述

11.2.2 back-ldap配置

11.2.3 进一步的信息



11.3 LDIF

11.3.1 概述

11.3.2 back-ldif配置

11.3.3 进一步的信息



11.4 LMDB

11.4.1 概述

11.4.2 back-mdb配置

11.4.3 进一步的信息



11.5 Metadirectory

11.5.1 概述

11.5.2 back-meta配置

11.5.3 进一步的信息



11.6 监控

11.6.1 概述

11.6.2 back-monitor配置

11.6.3 进一步的信息



11.7 零

11.7.1 概述

11.7.2 back-null配置

11.7.3 进一步的信息



11.8 Passwd

11.8.1 概述

11.8.2 back-passwd配置

11.8.3 进一步的信息



11.9 Perl /壳

11.9.1 概述

11.9.2 back-perl /底壳配置

11.9.3 进一步的信息



11.10 继电器

11.10.1 概述

11.10.2 返继电器配置

11.10.3 进一步的信息



11.11 SQL

11.11.1 概述

11.11.2 back-sql配置

11.11.3。进一步的信息