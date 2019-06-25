    OpenShift on Red Hat - take One



    I followed these instructions ( more or less )

    How To Install EPEL Repo on a CentOS and RHEL 7.x - nixCraft
    How do I install the extra repositories such as Fedora EPEL repo on a Red Hat Enterprise Linux server version 7.x or…www.cyberciti.biz
    Install EPEL repo first
    cd /tmp
     wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
     ls *.rpm
     yum install epel-release-latest-7.noarch.rpm
    [root@IoTFoodTrack tmp]# yum repolist
    Loaded plugins: product-id, search-disabled-repos, subscription-manager
    epel/x86_64/metalink | 10 kB 00:00:00
    epel | 5.4 kB 00:00:00
    rhel-7-server-extras-rpms | 2.0 kB 00:00:00
    rhel-7-server-optional-rpms | 1.8 kB 00:00:00
    rhel-7-server-rpms | 2.0 kB 00:00:00
    rhel-7-server-supplementary-rpms | 2.0 kB 00:00:00
    (1/3): epel/x86_64/group_gz | 88 kB 00:00:00
    (2/3): epel/x86_64/updateinfo | 977 kB 00:00:00
    (3/3): epel/x86_64/primary_db | 6.7 MB 00:00:00
    repo id repo name status
    docker-ce-stable/x86_64 Docker CE Stable - x86_64 43
    epel/x86_64 Extra Packages for Enterprise Linux 7 - x86_64 13,232
    kubernetes Kubernetes 360
    !rhel-7-server-extras-rpms/x86_64 Red Hat Enterprise Linux 7 Server - Extras (RPMs) 1,113
    !rhel-7-server-optional-rpms/7Server/x86_64 Red Hat Enterprise Linux 7 Server - Optional (RPMs) 17,777
    !rhel-7-server-rpms/7Server/x86_64 Red Hat Enterprise Linux 7 Server (RPMs) 24,346
    !rhel-7-server-supplementary-rpms/7Server/x86_64 Red Hat Enterprise Linux 7 Server - Supplementary (RPMs) 322
    repolist: 57,193
    $> ssh-keygen
    Check your server can indeed run Openshift?
    [root@IoTFoodTrack .ssh]# subscription-manager list - available - matches '*OpenShift*'
    …
    Red Hat CodeReady Workspaces for OpenShift
    Red Hat OpenShift Container Platform
    Red Hat OpenShift Service Mesh
    …
    SKU: RC1257407
    Contract: 11703931
    Pool ID: 2c9301526852de6c01685302a38c0a25
    Provides Management: Yes
    Available: 233801
    Suggested: 0
    Service Level: Premium
    Service Type: L1-L3
    Subscription Type: Stackable
    Starts: 01/01/2019
    Ends: 01/16/2020
    System Type: Physical
    Great! We can!!!
    In the output for the previous command, find the pool ID for an OpenShift Container Platform subscription and attach it:
    Now copy the
    Pool ID: 2c9301526852de6c01685302a38c0a25
    $> subscription-manager attach --pool=2c9301526852de6c01685302a38c0a25
    Successfully attached a subscription for: Red Hat Satellite - Add-Ons for Providers
    1 local certificate has been deleted.
    Now Disable all yum repositories:
    Disable all the enabled RHSM repositories:
    subscription-manager repos --disable="*"
    Repository 'rhel-ha-for-rhel-7-server-eus-rpms' is disabled for this system.
    Repository 'rhel-server-rhscl-7-rpms' is disabled for this system.
    Repository 'rhel-7-server-optional-rpms' is disabled for this system.
    Repository 'rhel-7-server-eus-optional-rpms' is disabled for this system.
    Repository 'rhel-7-server-rh-common-rpms' is disabled for this system.
    Repository 'rhel-7-server-eus-rpms' is disabled for this system.
    Repository 'rhel-ha-for-rhel-7-server-rpms' is disabled for this system.
    Repository 'rhel-rs-for-rhel-7-server-eus-rpms' is disabled for this system.
    Repository 'rhel-7-server-rpms' is disabled for this system.
    Repository 'rhel-7-server-supplementary-rpms' is disabled for this system.
    Repository 'rhel-7-server-extras-rpms' is disabled for this system.
    Repository 'rhel-7-server-eus-supplementary-rpms' is disabled for this system.
    List the remaining yum repositories and note their names under repo id, if any:
    $> yum repolist
    Loaded plugins: product-id, search-disabled-repos, subscription-manager
    repo id                                         repo name                                                               status
    docker-ce-stable/x86_64                         Docker CE Stable - x86_64                                                   43
    epel/x86_64                                     Extra Packages for Enterprise Linux 7 - x86_64                          13,232
    kubernetes                                      Kubernetes                                                                 360
    repolist: 13,635
    still more disabliing
    disable all repositories:
    yum-config-manager --disable \*
    #many where deleted! past my cli buffer - see appendix A
    try again
    yum repolist
    Loaded plugins: product-id, search-disabled-repos, subscription-manager
    repolist: 0
    Great no more repos! Bit disturbing but …
    Enable only the repositories required by OpenShift Container Platform 3.11.
    For cloud installations and on-premise installations on x86_64 servers, run the following command:

    subscription-manager repos \
        --enable="rhel-7-server-rpms" \
        --enable="rhel-7-server-extras-rpms" \
        --enable="rhel-7-server-ose-3.11-rpms" \
        --enable="rhel-7-server-ansible-2.6-rpms"
    Error: 'rhel-7-server-ose-3.11-rpms' does not match a valid repository ID. Use "subscription-manager repos --list" to see valid repositories.
    Error: 'rhel-7-server-ansible-2.6-rpms' does not match a valid repository ID. Use "subscription-manager repos --list" to see valid repositories.
    Repository 'rhel-7-server-rpms' is enabled for this system.
    Repository 'rhel-7-server-extras-rpms' is enabled for this system.
    eek two errors!
    no ose or ansible … googlling
    Ansible can now be found in rhel-7-server-extras-rpms along with the required dependencies.
    yum install ansible
    ...
    Installed:
    ansible.noarch 0:2.4.2.0-2.el7
    ...
    [root@IoTFoodTrack ~]# ansible --version
    ansible 2.4.2.0
    config file = /etc/ansible/ansible.cfg
    configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
    ansible python module location = /usr/lib/python2.7/site-packages/ansible
    executable location = /usr/bin/ansible
    python version = 2.7.5 (default, Sep 12 2018, 05:31:16) [GCC 4.8.5 20150623 (Red Hat 4.8.5-36)]
    yay Ansible is now installed!
    also SELinux needs to be enabled if need be
    see if it is
    [root@IoTFoodTrack ~]# sestatus
    SELinux status: enabled
    SELinuxfs mount: /sys/fs/selinux
    SELinux root directory: /etc/selinux
    Loaded policy name: targeted
    Current mode: permissive
    Mode from config file: enforcing
    Policy MLS status: enabled
    Policy deny_unknown status: allowed
    Max kernel policy version: 31
    if not edit /etc/ansible/ansible.cfg
    now back to ose!
    $> subscription-manager attach - pool=2c9301526852de6c01685302a38c0a25
    Successfully attached a subscription for: Red Hat Satellite - Add-Ons for Providers
    2 local certificates have been deleted.
    hmmm is this good yet????
    Well the sad truth is setting up OpenShift on Red Hat 7 is not easy and I an engineer for the last 25 years did not succeed :(
    But no worries! Onwards and upwards my next post will be how to setup OpenShift on Red Hat 8 [https://www.redhat.com/en/enterprise-linux-8 ]
    and will follow along with https://www.openshift.com/ and https://www.openshift.com/products/container-platform
    Just because something doesn't do what you planned it to do doesn't mean it's useless. - Thomas Edison (Inventor)
    Appendix A
    Debugging output - good reading for those of you who like TL;DR; or need something to put you to sleep :)
    List of disabled yum repos
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/epel-source/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/epel-source/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
    hdrdir = /var/cache/yum/x86_64/7Server/epel-source/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 21600
    metadata_expire_filter = read-only:present
    metalink = https://mirrors.fedoraproject.org/metalink?repo=epel-source-7&arch=x86_64
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Extra Packages for Enterprise Linux 7 - x86_64 - Source
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/epel-source
    pkgdir = /var/cache/yum/x86_64/7Server/epel-source/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert =
    sslclientcert =
    sslclientkey =
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = epel-source/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ===================================================== repo: epel-testing =====================================================
    [epel-testing]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl =
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/epel-testing
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/epel-testing/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/epel-testing/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
    hdrdir = /var/cache/yum/x86_64/7Server/epel-testing/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 21600
    metadata_expire_filter = read-only:present
    metalink = https://mirrors.fedoraproject.org/metalink?repo=testing-epel7&arch=x86_64
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Extra Packages for Enterprise Linux 7 - Testing - x86_64
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/epel-testing
    pkgdir = /var/cache/yum/x86_64/7Server/epel-testing/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert =
    sslclientcert =
    sslclientkey =
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = epel-testing/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ================================================ repo: epel-testing-debuginfo ================================================
    [epel-testing-debuginfo]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl =
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/epel-testing-debuginfo
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/epel-testing-debuginfo/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/epel-testing-debuginfo/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
    hdrdir = /var/cache/yum/x86_64/7Server/epel-testing-debuginfo/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 21600
    metadata_expire_filter = read-only:present
    metalink = https://mirrors.fedoraproject.org/metalink?repo=testing-debug-epel7&arch=x86_64
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Extra Packages for Enterprise Linux 7 - Testing - x86_64 - Debug
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/epel-testing-debuginfo
    pkgdir = /var/cache/yum/x86_64/7Server/epel-testing-debuginfo/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert =
    sslclientcert =
    sslclientkey =
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = epel-testing-debuginfo/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ================================================= repo: epel-testing-source ==================================================
    [epel-testing-source]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl =
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/epel-testing-source
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/epel-testing-source/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/epel-testing-source/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
    hdrdir = /var/cache/yum/x86_64/7Server/epel-testing-source/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 21600
    metadata_expire_filter = read-only:present
    metalink = https://mirrors.fedoraproject.org/metalink?repo=testing-source-epel7&arch=x86_64
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Extra Packages for Enterprise Linux 7 - Testing - x86_64 - Source
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/epel-testing-source
    pkgdir = /var/cache/yum/x86_64/7Server/epel-testing-source/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert =
    sslclientcert =
    sslclientkey =
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = epel-testing-source/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ====================================================== repo: kubernetes ======================================================
    [kubernetes]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/kubernetes
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = 0
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/kubernetes/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/kubernetes/gpgdir
    gpgkey = https://packages.cloud.google.com/yum/doc/yum-key.gpg,
     https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    hdrdir = /var/cache/yum/x86_64/7Server/kubernetes/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 21600
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Kubernetes
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/kubernetes
    pkgdir = /var/cache/yum/x86_64/7Server/kubernetes/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = True
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert =
    sslclientcert =
    sslclientkey =
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = kubernetes
    ui_repoid_vars = releasever,
     basearch
    username =
    =========================================== repo: rhel-7-server-eus-optional-rpms ============================================
    [rhel-7-server-eus-optional-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/eus/rhel/server/7/7Server/x86_64/optional/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-optional-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-optional-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-optional-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-optional-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Extended Update Support - Optional (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-optional-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-optional-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-eus-optional-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ================================================ repo: rhel-7-server-eus-rpms ================================================
    [rhel-7-server-eus-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/eus/rhel/server/7/7Server/x86_64/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Extended Update Support (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-eus-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ========================================= repo: rhel-7-server-eus-supplementary-rpms =========================================
    [rhel-7-server-eus-supplementary-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/eus/rhel/server/7/7Server/x86_64/supplementary/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-supplementary-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-supplementary-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-supplementary-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-supplementary-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Extended Update Support - Supplementary (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-eus-supplementary-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-eus-supplementary-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-eus-supplementary-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ============================================== repo: rhel-7-server-extras-rpms ===============================================
    [rhel-7-server-extras-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/extras/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-extras-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-extras-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-extras-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-extras-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Extras (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-extras-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-extras-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-extras-rpms/x86_64
    ui_repoid_vars = basearch
    username =
    ============================================= repo: rhel-7-server-optional-rpms ==============================================
    [rhel-7-server-optional-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/optional/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-optional-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-optional-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-optional-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-optional-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Optional (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-optional-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-optional-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-optional-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ============================================= repo: rhel-7-server-rh-common-rpms =============================================
    [rhel-7-server-rh-common-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/rh-common/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-rh-common-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rh-common-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rh-common-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-rh-common-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - RH Common (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rh-common-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-rh-common-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-rh-common-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ================================================== repo: rhel-7-server-rpms ==================================================
    [rhel-7-server-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    =========================================== repo: rhel-7-server-supplementary-rpms ===========================================
    [rhel-7-server-supplementary-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/supplementary/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-7-server-supplementary-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-supplementary-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-supplementary-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-7-server-supplementary-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux 7 Server - Supplementary (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-7-server-supplementary-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-7-server-supplementary-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-7-server-supplementary-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ========================================== repo: rhel-ha-for-rhel-7-server-eus-rpms ==========================================
    [rhel-ha-for-rhel-7-server-eus-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/eus/rhel/server/7/7Server/x86_64/highavailability/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux High Availability (for RHEL 7 Server) - Extended Update Support (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-eus-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-ha-for-rhel-7-server-eus-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ============================================ repo: rhel-ha-for-rhel-7-server-rpms ============================================
    [rhel-ha-for-rhel-7-server-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/highavailability/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux High Availability (for RHEL 7 Server) (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-ha-for-rhel-7-server-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-ha-for-rhel-7-server-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    ========================================== repo: rhel-rs-for-rhel-7-server-eus-rpms ==========================================
    [rhel-rs-for-rhel-7-server-eus-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/eus/rhel/server/7/7Server/x86_64/resilientstorage/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Enterprise Linux Resilient Storage (for RHEL 7 Server) - Extended Update Support (RPMs)
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-rs-for-rhel-7-server-eus-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-rs-for-rhel-7-server-eus-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
    =============================================== repo: rhel-server-rhscl-7-rpms ===============================================
    [rhel-server-rhscl-7-rpms]
    async = True
    bandwidth = 0
    base_persistdir = /var/lib/yum/repos/x86_64/7Server
    baseurl = https://rhncapdal1301.service.networklayer.com/pulp/repos/customer/Library/content/dist/rhel/server/7/7Server/x86_64/rhscl/1/os
    cache = 0
    cachedir = /var/cache/yum/x86_64/7Server/rhel-server-rhscl-7-rpms
    check_config_file_age = True
    compare_providers_priority = 80
    cost = 1000
    deltarpm_metadata_percentage = 100
    deltarpm_percentage =
    enabled = False
    enablegroups = True
    exclude =
    failovermethod = priority
    ftp_disable_epsv = False
    gpgcadir = /var/lib/yum/repos/x86_64/7Server/rhel-server-rhscl-7-rpms/gpgcadir
    gpgcakey =
    gpgcheck = True
    gpgdir = /var/lib/yum/repos/x86_64/7Server/rhel-server-rhscl-7-rpms/gpgdir
    gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release,
     file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
    hdrdir = /var/cache/yum/x86_64/7Server/rhel-server-rhscl-7-rpms/headers
    http_caching = all
    includepkgs =
    ip_resolve =
    keepalive = True
    keepcache = False
    mddownloadpolicy = sqlite
    mdpolicy = group:small
    mediaid =
    metadata_expire = 1
    metadata_expire_filter = read-only:present
    metalink =
    minrate = 0
    mirrorlist =
    mirrorlist_expire = 86400
    name = Red Hat Software Collections RPMs for Red Hat Enterprise Linux 7 Server
    old_base_cache_dir =
    password =
    persistdir = /var/lib/yum/repos/x86_64/7Server/rhel-server-rhscl-7-rpms
    pkgdir = /var/cache/yum/x86_64/7Server/rhel-server-rhscl-7-rpms/packages
    proxy = False
    proxy_dict =
    proxy_password =
    proxy_username =
    repo_gpgcheck = False
    retries = 10
    skip_if_unavailable = False
    ssl_check_cert_permissions = True
    sslcacert = /etc/rhsm/ca/katello-server-ca.pem
    sslclientcert = /etc/pki/entitlement/2066740905472802974.pem
    sslclientkey = /etc/pki/entitlement/2066740905472802974-key.pem
    sslverify = True
    throttle = 0
    timeout = 30.0
    ui_id = rhel-server-rhscl-7-rpms/7Server/x86_64
    ui_repoid_vars = releasever,
     basearch
    username =
