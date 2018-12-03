# Extra Configuration Options
This document intends to be an exhaustive list of all possible configuration parameters that can be placed in the `config.yaml` file.

## accept_oracle_bcl_terms
Accepting this means the entity running the installer accepts the Oracle Binary Code License Agreement and can proceed to install Java SE JDK. Otherwise the installation will fail.

## os\_configuration

### docker configuration
These keys relate to the configuration of the docker engine
* **docker_storage_driver** (*default*) : What storage driver to utilize for the
  docker storage engine. If omitted, the storage driver will be dynamically determined
  based on the OS distribution (aufs for Debian distros, overlay for Redhat derivatives).
  Primarily useful for switching back to devicemapper with loop-lvm mode. Supported options
  ``devicemapper``, ``aufs`` (ubuntu only), and ``overlay``.

### ssl
These keys define the SSL options used by nginx
* **enabled** (*false*) : whether SSL is enabled. If this is not present or
  is false, none of the other options in this section are used.
* **ca_file_name** : if provided, specifies the path to a certificate which lists
  the trusted certificate chain, so that clients can be authenticated. If
  this variable is specified, the variables `app_configuration.USER_AUTH_PKI_HEADER_NAME`
  and pki_header_name must also be defined.
* **pki_header_name** : must be provided if **ca_file_name** is provided. This
  specifies the name of the header that nginx will set after authenticating
  a user's certificate. It must match the value of `app_configuration.USER_AUTH_PKI_HEADER_NAME`
* **cert_file_name** : the name of the file used as the certificate for this
  server. This file must exist in the certs directory of the install directory
* **key_file_name** : the name of the file with the key that accompanies the
  cert file. This file must exist in the certs directory of the install directory
* **password_file** : (optional) Name of password file to be used with certificate.
  This file must exist in the certs directory of the install directory

### prediction\_ssl
These keys are used to configure SSL on the nginx server sitting in front of
the dedicated prediction servers. They are configured separately from the SSL
settings for the main DataRobot application.
* **enabled** (*false*) : whether SSL is enabled. If this is not present or is
  false, none of the other options in this section are used
* **ca_file_name** : if provided, specifies the path to a certificate which lists
  the trusted certificate chain, so that clients can be authenticated. All requests
  coming to this server will need to have certificates that authenticate correctly
  with this Certificate Authority. As opposed to the DR SSL config,since the
  prediction API applies another layer of authentication (via tokens) this setting
  does not result in a header being set - no pki_header_name is used for
  dedicated predictions.
* **cert_file_name** : the name of the file used as the certificate for this
  server. This file must exist in the certs directory of the install directory
* **key_file_name** : the name of the file with the key that accompanies the
  cert file. This file must exist in the certs directory of the install directory
* **password_file** : (optional) Name of password file to be used with certificate.
  This file must exist in the certs directory of the install directory

### code\_signing
These keys are used to configure DataRobot model code-signing feature. They
are basically a bunch of OpenSSL certificates used by DataRobot application,
not by Nginx. All keys (except for *enabled*) contain filenames and must exist
in the code-signing directory of the install directory.

* **enabled** (*false*): whether code-signing is enabled. If this is not present
  or is false, none of the other options in this section are used.
* **key_file_name**: the name of the file with the key that is used to sign
  exported datarobot models.
* **cert_file_name**: the name of the file used as the certificate to verify
  signatures made by the key.
* **ca_file_name**: specifies the path to a certificate which lists the trusted
  certificate chain, so that imported models can be checked whether they are
  trusted or not.
* **crl_file_name**: specifies the path to a certificate which lists all
  revoked certificates, so that we won't trust keys that are compromised.

### nfs
If customer-configured Network File Storage (or another distributed, mountable volume system) is used
to share `/opt/datarobot/data` across multiple edgenodes, writable by the `datarobot` user,
the `os_configuration.nfs` section can be configured with these keys:

* **enabled** (*false)*: Set to `true` to specify that `/opt/datarobot/data` is a NFS volume writable by datarobot user.

### secrets_enforced
Set to `True` in order to enforce password protection of databases. Passwords
will be generated automatically on `recreate-containers` if not already set.

### ide\_limit\_network
Set to `True` in order to limit network connectivity within IDE
sessions with iptables.  Note that this will create a crontab entry
with user `root`. Please execute `make bootstrap-cluster` after
changing the value.

### web\_api\_ip
When a hostname or non-IPv4 address is used for host running `internalapi` service, or
the `internalapi` service is behind a load balancer with a DNS record, set this to a valid
IPv4 address for the `internalapi` endpoint.

### webserver
Parameters for customization of NGINX webserver proxying requests for the DataRobot application services. The `os_configuration.webserver` section can be configured with these keys:

* **http_port** (*80*): HTTP port to use for nginx server. Defaults to 80. If `privileged` is set to false,
this must be set to a higher port (e.g. `8080`).
* **https_port** (*443*): HTTPs port to use for nginx server if SSL is used. Defaults to 443. If `privileged`
is set to false, this must be set to a higher port (e.g. `8443`).
* **privileged** (*true*): whether to run nginx as a privileged user. Defaults to true, and nginx service will
run as `admin_user`. When set to false, it runs as `user`.
* **upload_connection_timeout** (*600*): read timeout for responses from the application upload service which my occur when the file to process is too big (in seconds).

## app\_configuration
The following keys can be put in the `app_configuration` section of either the root-level dictionary or a host-specific `app_configuration`.
Key names are in **bold**, followed by their (*default*) settings, followed by a description of the key.
Keys are grouped into logically related sections.

### Queue settings

* **JOB_RETRY_IDLE_INTERVAL** (*300*): Number of seconds between job retry service scan iterations.
* **JOB_RETRY_MAX_RETRIES** (*10*): Maximum number of retry attempts per job.
* **JOB_RETRY_COUNTER_TTL** (*900*): TTL of retry attempts counter (in seconds).

### Worker settings
**NOTE** These settings do not affect Hadoop workers.
* **SECURE_WORKER_FIT_WORKERS** (*2*): Number of modeling workers to run per `secureworker` container.
* **MAX_N_JOBS_MULTIPROCESSING** (*4*): Maximum number of concurrent subprocesses a modeling worker can run.
* **MAX_N_JOBS_MULTITHREADING** (*4*): Maximum number of threads a modeling worker can run. Set to a lower number to reduce modeling worker CPU utilization.
* **EDA_WORKERS** (*1*): Number of EDA workers to run per `execmanager` container.

### File storage settings
* **FILE_STORAGE_TYPE** (*gluster_api*): For Cloudera installations set to `hdfs` (WebHDFS storage driver) or `hdfs3` (native HDFS storage driver). For Dockerized installations, set to `gluster_api` (Dockerized Gluster storage) or `s3` (AWS S3 storage).
* **FILE_STORAGE_PREFIX** : Represents the prefix applied to all paths in the file storage medium after the root path.
* **CACHE_LIMIT** (*2147483648*): Maximum size in bytes of the cache directory. The default location of this directory is `/opt/datarobot/data/app_data/cache`. The DataRobot application will remove older files from the cache when it writes data that would otherwise exceed this limit. **NOTE:** Must be smaller than the free space on the disk at all times.

### Log settings
* **DEBUG** (*False*): Set to `True` to enable debug logging for `datasetsservice` services.
* **LOGGING_LEVELNAME** (*INFO*): Controls log level of main application. Options are DEBUG, INFO, WARN, ERROR.

### Instrumentation settings
DataRobot components can optionally have statsd instrumenation enabled for performance monitoring and profiling.

* **PROFILE_DEFAULT_ALL** (*False*): Set to `True` when enabling statsd instrumentation.
* **PROFILE_USE_STATSD** (*False*): Set to `True` when enabling statsd instrumentation.
* **STATSD_ENABLED** (*False*): Set to `True` when enabling statsd instrumentation.
* **STATSD_NAME** (*unknown*): Set a custom name for your hosts metrics to identify your cluster.
* **STATSD_HOST** : Set to the hostname or IP address of the node you want to send statsd metrics to.
* **UWSGI_CARBON** : Set to the hostname or IP address of the Carbon Cache instance to send uwsgi metrics to.


### User authentication settings
Set these values to configure an LDAP authentication provider for DataRobot application accounts.

* **USER_AUTH_TYPE** (*internal*): Set to `ldap` or `ldapsearch` to enable LDAP authentication integration. `internal` configures DataRobot to use its own authentication system. `ldap` doesn't require a service LDAP user, but it can't be used with some LDAP configurations. `ldapsearch` covers all the uses cases of `ldap`, but it requires a service LDAP user (which can be configured via **USER_AUTH_LDAP_BIND_{DN,PASSWORD}**). Please see [LDAP Auth Deploy / Troubleshoot Guide](https://datarobot.atlassian.net/wiki/pages/viewpage.action?pageId=101744654) for details.
* **USER_AUTH_LDAP_URI** : URI of LDAP server Eg. `ldap://1.2.3.4:389`
* **USER_AUTH_LDAP_REQUIRED_GROUP** : DN of an LDAP group required for accessing DataRobot.
* **USER_AUTH_LDAP_REQUIRED_GROUP_MEMBER_ATTR** (*member*) : Name of LDAP attribute used to check group membership. Only used if **USER_AUTH_LDAP_REQUIRED_GROUP** set.
* **USER_AUTH_LDAP_DIST_NAME_TEMPLATE** : `ldap` mode - template for LDAP Dist Names. Eg. `mail=$username,cn=users,dc=datarobot,dc=com`.
* **USER_AUTH_LDAP_BIND_DN** : `ldapsearch` mode - DN of the service LDAP account.
* **USER_AUTH_LDAP_BIND_PASSWORD** : `ldapsearch` mode - PASSWORD of the service LDAP account.
* **USER_AUTH_LDAP_SEARCH_BASE_DN** : `ldapsearch` mode - LDAP node that contains all the DR users. Eg. `cn=users,cn=accounts,dc=datarobot,dc=com`.
* **USER_AUTH_LDAP_SEARCH_SCOPE** (*SUBTREE*): `ldapsearch` mode - LDAP search scope (`ONELEVEL` or `SUBTREE`).
* **USER_AUTH_LDAP_SEARCH_FILTER** (`(cn=$username)`): `ldapsearch` mode - LDAP search query template.

Set these values to configure authentication via Public Key certificates. Keep in mind that these
are just the application configuration variables - in order to enable PKI you will also need
to configure the `os_configuration.ssl` variables so that nginx can correctly authenticate
clients.

* **USER_AUTH_PKI_ENABLED** (*false*): boolean, enables PKI auth
* **USER_AUTH_PKI_HEADER_NAME** (*X-Ssl-Client-S-Dn*): string, recommend "X-Ssl-Client-S-Dn".
  This is the name of the header that nginx will set after authenticating clients. This
  value must match the value of `os_configuration.ssl.pki_header_name`.
* **USER_AUTH_PKI_AUTO_CREATE_ACCOUNT** (*false*): boolean. if enabled, any authenticated
  client will get a new account created on DataRobot. Otherwise, an admin must
  create the account before the user can access the system at all.
* **USER_AUTH_PKI_MAPPING** List of fields to be copied from the certificate subject to the local
  DR profile in the format `{local_field_name: subject_field_name}` i.e. `{'username': 'emailAddress', 'display_name': 'CN'}`

### Hadoop settings
These settings are specific to Hadoop installations.

* **ENABLE_HADOOP_DEPLOYMENT** (*False*): Set to `True` to turn on Hadoop-related features.
* **SECURE_WORKER_USER_TASK_IMAGE** (*sw-model-v1.0*): Set this to `subprocess` on Hadoop systems.

**WARNING:**
You should not need to modify the below settings yourself, even if the defaults do not match your system.
The installation process should set them appropriately in your containers via the properties file.
*Do not modify them unless you know what you are doing*.

* **KERBEROS_ENABLE** (*False*): Whether Kerberos authentication is used when connecting to HDFS.
* **DATAROBOT_CONFIG** (*/opt/datarobot/etc/hadoop/datarobot-defaults.conf*): The path inside the application container to find the DataRobot properties file. Change this if you want to use an alternative file without modifying the existing file.
* **KRB5_CONFIG** (*/opt/datarobot/etc/hadoop/krb5.conf*): The path inside the application containers to find Kerberos settings.
* **KERBEROS_KEYTAB_FILE** (*/opt/datarobot/etc/hadoop/datarobot.keytab*): The path inside the application containers to find the Kerberos keytab file.
* **KERBEROS_PRINCIPAL** : Name of the Kerberos principal your containers should use for authentication.
* **KERBEROS_KINIT_LOCATION** (*/usr/bin/kinit*): The location of the kinit binary in your containers.
* **KERBEROS_TGT_LIFETIME** (*86400*): The lifetime of Kerberos tickets.
* **WEBHDFS_IS_SECURE** (*False*): Whether WEBHDFS connections are secured by Kerberos authentication.
* **WEBHDFS_HOST** : The location of the WEBHDFS host in the form `http://host:port`.
* **WEBHDFS_IMPERSONATION_FIELD** : User to impersonate.
* **WEBHDFS_IMPERSONATION_FIELD_TO_LOWER** (*True*): Make the value of the impersonation field lowercase.
* **WEBHDFS_STORAGE_DIR** : The root of the path used for storing files in HDFS.
* **WEBHDFS_DEFAULT_USER** (*datarobot*): The user to impersonate on a cluster not secured by Kerberos.
* **YARN_REST_API_HOST** : Location of the YARN REST API host.
* **VW_PATH** (*/opt/cloudera/parcels/DataRobot/bin/vw*): Location of the Vowpal Wabbit binary in Cloudera parcels.

## Prediction settings
Only set these settings on dedicated prediction nodes.

* **dedicated_prediction_server** (*False*): Set to `True` on dedicated prediction servers to enable high-performance prediction configuration.


## drenv\_override
The following keys can be put in the `drenv_override` sub-dictionary of any `app_configuration` dictionary.
This section is not likely to be an exhaustive list of all configuration options available. However, the most relevant will be recorded here.

### SMTP Email configuration
These settings configure email for application support integration.
They allow users of your application to send support messages to your organization.

* **SMTP_ADDRESS** : Address of the SMTP service that runs your organization's mail, _eg._ `smtp.gmail.com`.
* **SMTP_PORT** (*587*): Port to use to connect to your SMTP server.
* **SMTP_MODE** (*SMTP_STARTTLS*): SMTP mode for your network. Options are `SMTP`, `SMTP_SSL`, or `SMTP_STARTTLS`.
* **SMTP_USER** (*''*): Username to use when connecting to the SMTP server if authentication is required.  If no authentication is needed to connect, should be left unspecified.
* **SMTP_PASSWORD** (*''*): Password to use for authentication, if required.  If no authentication is needed to connect, should be left unspecified.
* **HTML_EMAILS_ENABLED** (*true*): Whether to send HTML emails (default) or only plaintext.  HTML emails look nicer, but include images that require internet access when reading the emails.
* **DEFAULT_SENDER** (*support@datarobot.com*): Email address for the application to send email as, _eg._ `megaphone@customer-domain.com`.
* **DEFAULT_APPROVER** (*support@datarobot.com*): The email address of the person responsible for adding new accounts.
* **DEFAULT_CFDS_RECEIVER** (*cfds@datarobot.com*): The email address that should be notified when new users are added to the application.
* **DEFAULT_SUPPORT** (*support@datarobot.com*): The address that users will be told to contact for application support.
* **CONTRACTS_EMAIL** (*clickthrough@datarobot.com*): The email address that should be notified when users accept or decline the subscription agreement.

### Dedicated prediction servers
*  **WORKER_MODEL_CACHE** (*16*): Models to cache in memory at once. Tune this for your use case.
*  **MODEL_CACHE_MODE** (*LRU*): Select the cache mode for model caching. Options are `LRU` and `latest`.

The following settings should be set to `True` on dedicated prediction nodes for optimum performance.

**WARNING:**
These settings must only be set to `True` on dedicated high-performance prediction nodes.
If they are set on non-prediction services the application will not function correctly.

*  **CACHE_DB_RESULT** (*False*): Enables caching of database query results, giving improved performance for dedicated prediction servers.

### File upload size
These settings determine the maximum file ingest size for your cluster. Note that 11 Gigabytes (the default setting) is a hard upper limit on the amount of data that DataRobot is currently able to process.

* **MAX_CONTENT_LENGTH** (*11811160064*): Maximum size of file that can be uploaded in bytes.
* **MAX_DECOMPRESSED_SIZE** (*5368709120*): Maximum decompressed size of file that can be uploaded in bytes.
* **MAX_PREDICTION_CONTENT_LENGTH** (*1073741824*): Maximum size of files for batch predictions in bytes.

### Activity Monitor
These settings enable audit logging and activity monitor UI.

* **AUDIT_LOGS_LOGSTASH_HOST** (*empty*): Hostname of logstash server that accepts audit logs for writing them to Mongo and on disk. The application doesn’t send audit logs to logstash if this variable is missing or empty.
* **AUDIT_LOGS_LOGSTASH_PORT** (*5544*): Logstash port number listening to audit logs.

### SELinux
* **ENABLE_SELINUX** (*False*): Set to `True` to install the application in an SELinux Enforcing environment.

### AWS access
Set these keys if you need to use AWS keys to access S3-based file storage (see `app_configuration.FILE_STORAGE_TYPE`). IAM role-based storage access is not yet available.

* **AWS_ACCESS_KEY_ID** : Access key ID for the account you want to use to connect to S3 storage.
* **AWS_SECRET_ACCESS_KEY** : Secret access key for authenticating your AWS account.
* **S3_BUCKET** : Name of the S3 bucket to store DataRobot application files in. Your access key ID must belong to an account that has write, read, and list permissions on this bucket.

### Password policies
These settings can be used to set password policies when DataRobot internal authentication is used.

* **PASSWORD_EXPIRATION_TIME** (*0*): Users' passwords will expire this many days after they are set. When a user attempts to login with an expired password, they will be promped to set a new password before they can access DataRobot. If this is less than or equal to zero, then passwords do not expire.
* **PASSWORD_EXPIRATION_WARNING_TIME** (*15*): Users will be shown warnings that their password is about to expire this many days in advance of its expiration.
* **PASSWORD_HISTORY_LENGTH** (*5*): If password expiration is enabled, users will be prevented from reusing this many of their previous passwords.

### Ingest methods
These settings can be used to restrict the user's ability to upload data from different sources into the application. These settings will only restrict project creation and prediction uploads from these sources:

* **ALLOW_INGEST_METHOD_LOCAL** (*True*): Enables the user's ability to upload local files into the application.
* **ALLOW_INGEST_METHOD_DATA_SOURCE** (*True*):Enables the user's ability to upload data from ODBC and JDBC sources into the application.
* **ALLOW_INGEST_METHOD_URL** (*True*): Enables the user's ability to upload data from an URL into the application.

### Model Monitoring
These settings can be used to change amount of data reported to Model Monitoring, that can be used to limit performance hit from logging and processing stats:

* **PREDICTION_API_MONITOR_USAGE_ENABLED** (*False*): Enables tracking of service stats (response codes, response time, etc).
* **PREDICTION_API_MONITOR_RESULT_ENABLED** (*False*): Enables tracking of prediction results.
* **PREDICTION_API_MONITOR_RAW_ENABLED** (*False*): Enables tracking of prediction features.
* **PREDICTION_API_MONITOR_RAW_MAX_FEATURES** (*10*): Maximum number of the most important features to report.
* **ENABLE_MODEL_DEPLOYMENTS** (*True*): Enables tracking of all stats (regardless of the values of previous options).

Note, that these settings define Prediction API server level configuration. Values of **PREDICTION_API_MONITOR_RESULT_ENABLED** and **PREDICTION_API_MONITOR_RAW_ENABLED** can be overriden by users on a model deployment level via Public API (e.g. if they want to minimize the impact of model management instrumentation on prediction latency for a given model deployment).

### Miscellaneous
* **ALLOW_SELF_SIGNED_CERTS** (*False*): Set to `True` to disable SSL certificate verification for requests made by the application to the web server. Set this if you have SSL encryption enabled on your cluster and are using certificates that are not signed by a globally trusted Certificate Authority.
* **STREAM_FORMATTER_TYPE** (*json*): Format for DataRobot application logs. Set to `plain` for more human-readable logs. (Note: some components will not respect this setting)
* **NEXT_STEPS_WORKERS** (*100*): Number of workers available for queue management per `execmanager` service. You shouldn't need to increase this unless you have a massive number (100's) of projects running in parallel.
* **EXTERNAL_WEB_SERVER_URL** : The base url users use to access the webserver, e.g. "https://app.datarobot.com".  Set this variable if it differs from the internally configured hostname, for example if the web server is behind a load balancer.
* **EXTERNAL_WEB_SERVER_URL_FORCED** (*False*): Force usage of *EXTERNAL_WEB_SERVER_URL* to generate urls and never use host name supplied in HTTP headers.
* **ENABLE_PROJECT_SHARING** (*True*): Enable users to share projects with each other. By default multiple users are able to work on the same project through project sharing using the *Share* button in the application header. Set `False` to disable project sharing and hide the button.


## Root-level keys
The following keys are very rarely used.
They should be placed at the root level of the `config.yaml` dictionary (_ie._ zero indentation).

* **accept_oracle_bcl_terms** (*false*): Accept Oracle Binary Code License Agreement and install Java SE Platform Products.
* **docker_version** (*17.03.1-ce*): Version of Docker to use. Will break `make bootstrap-cluster` if this version is not installed on all nodes.
* **docker_py_version** (*1.9.0*): Version of docker-py library to use for Ansible to control Docker containers. Other versions of docker-py are not supported and may have compatibility issues between Docker and Ansible.
* **docker_group_name** (*docker*): Name of the group that `/var/run/docker.sock` is owned by. The DataRobot user will be added to this group by `make bootstrap-cluster` if it is run. Users must be in this group to issue Docker commands.
