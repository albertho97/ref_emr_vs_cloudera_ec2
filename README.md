1. **Table comparison** for EMR vs Cloudera on EC2 (AWS)
2. **Pros and Cons** if switching to EMR under cost cutting situation
3. How to mitigate the &quot;Cons&quot; when switching to EMR. List out the **shortfall** for the things that cannot be achieved and the workaround for them.

|   | AWS EMR | **Cloudera on EC2** |
| --- | --- | --- |
| License Cost Impact | No licenses cost, just EC2 + EMR rate costs | Enterprise licenses + EC2 costs |
| Scalability | Can do auto scaling + spot instance | Due to license per node, not so easy to do auto scaling |
| Storage Impact Filesystem (HDFS) and S3 | S3 (EMRFS), Compute and Storage are loosely decoupled. S3 is the primary storage for the entire data lake. HDFS will become buffer storage. Much lower storage cost on S3 than EBS. [Link](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-file-systems.html) | HDFS is the primary storage, require EBS disk permanently provisioned. Compute and Storage cannot be decoupled |
| Controller of Cluster | Ganglia UI, a web console for monitoring, and managing cluster will be through AWS EMR portal, UI functionality are much less than Cloudera Manager, some orchestration tasks require manual scripting | Cloudera Manager UI, very flexible controller to manage the entire cluster |
| AWS Ecosystem integration | Better integrate with other AWS components, such as Kinesis, S3 (EMRFS) and IAM (Identity and Access Management) | Cloudera also has S3Connector, it can only using S3a:// protocol, instead of s3:// (EMRFS), which has different compatibility level, certain DDL cannot be done on s3a:// |
| Security | AWS IAM + Ranger + AWS KMS. It covers security, it may not have as fine-grained control as cloudera navigator. [Link](https://aws.amazon.com/blogs/big-data/implementing-authorization-and-auditing-using-apache-ranger-on-amazon-emr/) | Cloudera Navigator + Trustee KMS server + Sentry to protects the clusters, which has more fine-grain level of controls |
| Security – Ranger vs Sentry | Though it is from beginning of 2016, at least is done by a third party. Scroll to the bottom, and click on the image for the comparison table. [Link](https://blogs.informatica.com/2016/01/16/securing-sensitive-information-big-data-world/#fbid=lX_Bp0k88NQ) |   |
| Audit | Ranger logs HDFS and Hive access + AWS Cloudtrail for any API call activity to the cluster. In comparison to Navigator, the key question is how fast Ranger in the community is growing, to what extent what hadoop ecosystem it is covering. [Link](http://ranger.apache.org/faq.html) | Cloudera Navigator logs everything happen to the cluster + logging to HDFS, Hive, Impala, etc. It has a comprehensive architecture, as the cloudera agent talks to the audit server. [Link](https://www.cloudera.com/documentation/enterprise/5-11-x/topics/cn_iu_audit_arch.html) |
| Encryption | At-rest data encryption. In-transit data encryption. HDFS transparent encryption. Server-side / client side encrpytion. [Link](http://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-data-encryption-options.html) | Also has full set of encryption functionality, using Cloudera Navigator, Trustee Key Server, HSM VM, etc |
| Interactive Query End user Authentication and Authorization | EMR does not support Impala natively, instead it relies on Presto and Hive. Ranger supports Hive on Tez at the moment, so Hive will be the main query engine used for data explorer / scientist. Until Presto has better authorization mechanism for interactive query, or need to investigate into Data virtualization like Denodo for security governance. Like slide 44 (Data Lake security in below slideshare) | Because Sentry can cover Impala&#39;s access control, it is ready for data explorer / scientist to safely consume the services. |
| High Availability | Currently no HA for Master node, AWS mention it will be available in Q4 2017. For other data nodes, it can recover crashed node automatically | Cloudera Manager have feature to do NameNode HA, and HA for other Hadoop components |
| Backup and Restore | EMR relies on S3 ( EMRFS ), and S3 can do versioning if required. For Hive metadata, it will be stored in external MySQL database, which we can do daily backup or restore. | Cloudera Manager can do HDFS snapshotting. Using external database to store metadata of Hive as well. |
| DR | EMRFS is natively sitting on S3, S3 is regionally available across AZs, meaning no copy data is needed during DR, what will happen is to re-spin up a EMR cluster another AZ, as the Hive tables are all external tables sitting in S3. | Using Cloudera BDR, replicating data to S3, during DR, replicate back from S3 to HDFS. The process is managed through Cloudera Manager. |
| Informatica Compatibility | Compatible with EMR. [Link](https://kb.informatica.com/h2l/HowTo%20Library/1/0958-Configuring_BDM_in_Amazon_Cloud-H2L.pdf) | Compatible with Cloudera |
| BI tools Compatibility | Hive has JDBC / ODBC driver for connectivity | Impala has JDBC / ODBC driver for connectivity |
| Kafka vs Kinesis | AWS provide an alternative to Kafka, which is Kinesis and it is a PaaS, which can integrate with EMR. E.g. Inside a Spark streaming, you can refer to an external table which is Kinesis stream through a driver written by AWS. [Link](https://aws.amazon.com/blogs/big-data/optimize-spark-streaming-to-efficiently-process-amazon-kinesis-streams/) | Cloudera Manager support running Kafka through managing the VMs |
| Sizing of instances Impact | Following EMR guidelines, in general the instance sizes are smaller because of using EMRFS, as the maintenance of filesystem are mainly on S3 now, HDFS are there but only acts as temporary storage. [Link](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-instances-guidelines.html)  | Cloudera has its guideline of sizing the VM instances, in general slightly higher VM specs are required. |
| OS Impact | EMR use Amazon Linux 64bit by default, as it is a semi-PaaS from infrastructure perspective, it also supports custom AMI, but the custom AMI has to be based on Amazon Linux 64bit. So the hardening will be performed on top of Amazon Linux 64bit. [Link](http://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-custom-ami.html)  | It will be based on Redhat linux |
| Version upgrade approach | Will need to use a &quot;Blue-green&quot; update method with Route 53, having a global DNS mapping to the cluster endpoint, then spill up new cluster with new version, and change global DNS to point to new cluster, and destroy old one. For more details on EMR endpoint using Route 53, here is one of the blog talking about it, with private DNS server and private zone record. [Link](https://aws.amazon.com/blogs/big-data/launching-and-running-an-amazon-emr-cluster-in-your-vpc-part-2-custom-dns/) | Cloudera Manager has nicely built-in rolling upgrade feature. |
| How to leverage S3 storage more | EMR using s3:// (EMRFS) that is fully compatible with S3, as it is a proprietary driver by AWS. It can use S3 like a HDFS filesystem.  | Cloudera option, we are currently already leveraging S3 as cold archive as an external data store, but cannot use S3 as a HDFS replacement |
| AWS Future Component | AWS Redshift spectrum, directly query S3 storage data, provide JDBC / ODBC and standard database security. AWS Glue (preview), which is AWS version of ETL to do transformation |   |

**AWS re:Invent 2016: Deep Dive: Amazon EMR Best Practices &amp; Design Patterns**
[Link](https://www.slideshare.net/AmazonWebServices/aws-reinvent-2016-deep-dive-amazon-emr-best-practices-design-patterns-bdm401)

**Challenge for migrating to EMR from other Big Data technologies**
[Link](https://www.slideshare.net/AmazonWebServices/bda-302-deep-dive-on-migrating-big-data-workloads-to-amazon-emr)

**Pros for AWS EMR**

- Auto scaling cluster, use of spot instance. Because of compute and storage separation, can easily scale according to burst demand
- No license cost, included as part of using AWS benefit
- EMR is being treated as transient cluster, as all permanent data are stored on S3, therefore reducing EBS disk cost
- Better AWS Ecosystem integration
- DR failover time should be less than 45 mins, because all data are stored on S3 across AZs already

Cons for AWS EMR

- Losing data locality nature in Hadoop, using S3 instead of local HDFS, for some type of queries, there will be some performance drawback.
- For interactive query authentication / authorization, EMR use Ranger, which supports only Hive. Comparing to Impala with Sentry on cloudera, the performance difference become Impala vs Hive on Tez. Impala should be faster.
- High availability of Master Node, EMR does not yet provide HA for name node until Q4 2017, meaning all ETL transformation result should be saved back to S3, HDFS inside EMR can only be used as temporary and buffer storage
- Ease of use UI / controller **,** EMR portal UI is not as comprehensive as Cloudera Manager
- Security control UI, the EMR does not as fine grain level of security settings control as Cloudera Manager, instead it provides security configuration at launch wizard.

**Mitigation of some of the Cons for AWS EMR**

- Interactive query authentication / authorization
If Hive on Tez is not fast enough, will need to consider &quot;Redshift&quot; and &quot;Presto&quot;, and adding data virtualization layer like Denodo for authorization purpose. This will increase costs though.
- High availability
If costs allow, spin up another EMR cluster for interactive query, as all data are in S3 (EMRFS), and AWS allows multiple EMR cluster pointing to the same S3 data.


**Appendix – screenshots**

**Ganglia UI for monitoring EMR cluster**


**Ranger with EMR Architecture**


**Ranger – Audit logging**


**Ranger – RBAC access control for Hive / HDFS**


**EMRFS**


**Kinesis with Spark Streaming in EMR**


**EMR Encryption**


**EMR Spot Instance**


**EMR Auto scaling by metrics**
