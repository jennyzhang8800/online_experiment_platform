部署Shibboleth需要LDAP服务器,IDP服务器,以及在Gitlab,OpenEdX上分别部署SP.
====

更详细的文档以及配置文件可以参见[shibboleth仓库](https://github.com/rainymoon911/shibboleth-for-edx)

1.部署LDAP服务器.
====

1.1 安装OpenLDAP即可视化工具

[我们所使用的OpenLDAP镜像](http://www.turnkeylinux.org/openldap)

该镜像已包含phpldapadmin(方便网页端访问LDAP),若从其他渠道安装LDAP,请自行安装该工具。

1.2 利用eduperson.ldif创建模式eduPerson

    ldapadd -Y EXTERNAL -H ldapi:/// -f <path of eduperson.ldif>
    
1.3 登录管理员账号创建存储用户的结点,例如ou=Users,dc=openedx,dc=com

1.4 test_ldap.py可用于测试OpenLDAP是否正常工作(修改其中的ip,baseDN以及searchFilter参数,保持与IDP中的配置一致,
详细可参考shibboleth仓库中的配置文件)

1.5 createUser.ldif用于手动创建用户(修改其中的用户参数)


2.部署IDP
====

2.1 安装apache 以及 tomcat:

    apt-get install apache2
    a2enmod ssl
    a2enmod proxy_ajp

    sudo apt-get install tomcat6
    
[apache和tomcat的配置](https://wiki.shibboleth.net/confluence/display/SHIB2/IdPApacheTomcatPrepare) 
    
    vi /etc/hosts
    //add the following code
    127.0.0.1 idp.edx.org sp
    
    vi /tomcat6/apache2.conf
    //add the following code
    ServerName idp.edx.org
    
2.2 jdk(oracle jdk--officially recommended):

    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java7-installer
    
configure jdk

    update-alternatives --config java
    
    
2.3 idp:

我们系统间的交互是内部的,所以不使用https(所以将配置文件中的https改为了http)
(ps:SAML协议即使不使用https传输数据,也是有一定的安全性的)

2.3.1 安装idp,这里使用的版本是2.4.4

[download source](http://shibboleth.net/downloads/identity-provider/)
    
    download the source
    unzip shibboleth-identityprovider-2.4.4-bin.zip
    cd shibboleth-identityprovider-2.4.4
    JAVA_HOME=/usr/lib/jvm/java-7-oracle ./install.sh
    chown -R tomcat6:tomcat6 /opt/shibboleth-idp

访问idp.edx.org:8080//idp/profile/Status,若一切正常,可看到ok

如果出现错误,请查看日志:/opt/shibboleth-idp/logs/idp-process.log

[官方的问题解答列表](https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPTroubleshootingCommonErrors#NativeSPTroubleshootingCommonErrors-Unabletolocatemetadataforidentityprovider(https://identities.supervillain.edu/idp/shibboleth).)

idp的默认端口是8080(8443用于ECP),如果使用默认端口的话,配置文件中的idp.edx.org 需要替换为 idp.edx.org:8080



2.3.2 配置SP端的 metadata (sp的metadata生成,在安装sp的步骤中会提及)

(metadata在idp端以及sp端的名字可自行更改,但必须保持一致)

    scp sp-metadata.xml username@idp-ip:/opt/shibboleth-idp/metadata
    
    vi /conf/relying-party.xml
    //add the following code
    <metadata:MetadataProvider xsi:type="FilesystemMetadataProvider"
		xmlns="urn:mace:shibboleth:2.0:metadata" id="SPMETADATA"
		metadataFile="/opt/shibboleth-idp/metadata/sp-metadata.xml" />
		
2.3.3 配置LDAP验证

在配置前,请使用test_ldap.py保证LDAP正常工作

	vi handler.xml
	//add the following code
	<!--  Username/password login handler -->
	<ph:LoginHandler xsi:type="ph:UsernamePassword"
		jaasConfigurationLocation="file:///opt/shibboleth-idp/conf/login.config">
	<ph:AuthenticationMethod>urn:oasis:names:tc:SAML:2.0:ac:classes:
		PasswordProtectedTransport</ph:AuthenticationMethod>
	</ph:LoginHandler></code>
		
	vi attribute-resolver.xml
	//add the following code
	<resolver:DataConnector id="myLDAP" xsi:type="dc:LDAPDirectory"
        ldapURL="ldap://<your ldap url>" 
        baseDN="ou=Users,dc=openedx,dc=com" 
        principal="cn=admin,dc=openedx,dc=com"
        principalCredential="password">
        <dc:FilterTemplate>
            <![CDATA[
                (uid=$requestContext.principalName)
            ]]>
        </dc:FilterTemplate>
    	</resolver:DataConnector>
    	
    	vi attrbute-filter.xml
    	//add the following code
    	<afp:AttributeRule attributeID="email">
	    <afp:PermitValueRule xsi:type="basic:ANY" />
    	</afp:AttributeRule>

	<afp:AttributeRule attributeID="commonName">
	    <afp:PermitValueRule xsi:type="basic:ANY" />

修改attribute-resolver.xml中的ldapURL,baseDN,以及管理员名称,密码

3.在Gitlab上配置SP
====


    
