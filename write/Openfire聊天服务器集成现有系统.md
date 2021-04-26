# Openfire 聊天服务器集成现有系统

这里忽略安装以及配置Openfire数据库的过程，直接讲集成过程。

官方文档中提到的集成方法是在openfire\conf\openfire.xml文件中进行配置，但是实践过程中发现这种方式无效，官方论坛上提到，xml中的配置多半是无效的，还是要在数据库中进行配置，其实也就只是在Openfire中的一张表中配置罢了——ofproperty。

下面是安装配置好openfire之后，对ofproperty进行基本的验证、用户、用户组的集成，可根据自己的业务需求，直接修改set之后的变量值。字段的意思可以参照Openfire的官方文档来理解。

```sql
begin transaction
--clear existed field
delete from ofProperty where name in ('admin.authorizedJIDs', 'jdbcAuthProvider.passwordSQL', 'jdbcAuthProvider.passwordType',
    'jdbcProvider.connectionString', 'jdbcProvider.driver', 'jdbcUserProvider.allUsersSQL', 'jdbcUserProvider.emailField', 
    'jdbcUserProvider.loadUserSQL', 'jdbcUserProvider.nameField', 'jdbcUserProvider.userCountSQL', 'jdbcUserProvider.usernameField',
    'provider.admin.className', 'provider.auth.className', 'provider.group.className', 'provider.lockout.className', 'provider.securityAudit.className',
    'provider.user.className', 'provider.vcard.className', 'xmpp.auth.anonymous', 'xmpp.domain', 'xmpp.session.conflict-limit', 'xmpp.socket.ssl.active',
    'jdbcGroupProvider.groupCountSQL', 'jdbcGroupProvider.allGroupsSQL', 'jdbcGroupProvider.userGroupsSQL', 'jdbcGroupProvider.descriptionSQL',
    'jdbcGroupProvider.loadMembersSQL', 'jdbcGroupProvider.loadAdminsSQL');

--declare field values
declare @domain nvarchar(20);
declare @driver nvarchar(100);
declare @connect_str nvarchar(200);
declare @author nvarchar(50);

declare @pw_sql nvarchar(500);
declare @pw_type nvarchar(20);

declare @all_user_sql nvarchar(500);
declare @email_field nvarchar(20);
declare @name_field nvarchar(20);
declare @load_user_sql nvarchar(500);
declare @user_count_sql nvarchar(500);
declare @user_name_field nvarchar(20);

declare @group_count_sql nvarchar(500);
declare @all_group_sql nvarchar(500);
declare @user_group_sql nvarchar(500);
declare @description_sql nvarchar(500);
declare @load_members_sql nvarchar(500);
declare @load_admin_sql nvarchar(500);

set @domain = 'rjb11';
set @driver = 'org.postgresql.Driver';
set @connect_str = 'jdbc:postgresql://127.0.0.1:5432/myim?user=postgres&password=000000';
set @author = 'admin@gason';

set @pw_sql = 'select password from t_web_user where account=?';
set @pw_type = 'plain';

set @all_user_sql = 'select account from t_web_user';
set @email_field = 'email';
set @name_field = 'name';
set @load_user_sql = 'select account,email from t_web_user where account=?';
set @user_count_sql = 'select count(*) from t_web_user';
set @user_name_field = 'name';

set @group_count_sql = 'select count(*) from mygroups';
set @all_group_sql = 'select groupname from mygroups';
set @user_group_sql = 'select groupname from mygroupusers where username=?';
set @description_sql = 'select groupdescription from mygroups where groupname=?';
set @load_members_sql = 'select username from mygroupusers where groupname=? and isadmin=''N''';
set @load_admin_sql = 'select username from mygroupusers where groupname=? and isadmin=''Y''';

--insert settings
insert into ofproperty(name, propValue) values('provider.admin.className', 'org.jivesoftware.openfire.admin.DefaultAdminProvider');
insert into ofproperty(name, propValue) values('provider.auth.className', 'org.jivesoftware.openfire.auth.DefaultAuthProvider');
insert into ofproperty(name, propValue) values('provider.group.className', 'org.jivesoftware.openfire.group.DefaultGroupProvider');
insert into ofproperty(name, propValue) values('provider.lockout.className', 'org.jivesoftware.openfire.lockout.DefaultLockOutProvider');
insert into ofproperty(name, propValue) values('provider.securityAudit.className', 'org.jivesoftware.openfire.security.DefaultSecurityAuditProvider');
insert into ofproperty(name, propValue) values('provider.user.className', 'org.jivesoftware.openfire.user.DefaultUserProvider');
insert into ofproperty(name, propValue) values('provider.vcard.className', 'org.jivesoftware.openfire.vcard.DefaultVCardProvider');
insert into ofproperty(name, propValue) values('xmpp.auth.anonymous', 'true');
insert into ofproperty(name, propValue) values('xmpp.domain', @domain);
insert into ofproperty(name, propValue) values('xmpp.session.conflict-limit', '0');
insert into ofproperty(name, propValue) values('xmpp.socket.ssl.active', 'true');
                                        
insert into ofproperty(name, propValue) values('jdbcProvider.connectionString', @connect_str);
insert into ofproperty(name, propValue) values('jdbcProvider.driver', @driver);
insert into ofproperty(name, propValue) values('admin.authorizedJIDs', @author);                                           
insert into ofproperty(name, propValue) values('jdbcAuthProvider.passwordSQL', @pw_sql);
insert into ofproperty(name, propValue) values('jdbcAuthProvider.passwordType', @pw_type);

insert into ofproperty(name, propValue) values('jdbcUserProvider.allUsersSQL', @all_user_sql);
insert into ofproperty(name, propValue) values('jdbcUserProvider.emailField', @email_field);
insert into ofproperty(name, propValue) values('jdbcUserProvider.loadUserSQL', @load_user_sql);
insert into ofproperty(name, propValue) values('jdbcUserProvider.nameField', @name_field);
insert into ofproperty(name, propValue) values('jdbcUserProvider.userCountSQL', @user_count_sql);
insert into ofproperty(name, propValue) values('jdbcUserProvider.usernameField', @user_name_field);

insert into ofproperty(name, propValue) values('jdbcGroupProvider.groupCountSQL', @group_count_sql);
insert into ofproperty(name, propValue) values('jdbcGroupProvider.allGroupsSQL', @all_group_sql);
insert into ofproperty(name, propValue) values('jdbcGroupProvider.userGroupsSQL', @user_group_sql);
insert into ofproperty(name, propValue) values('jdbcGroupProvider.descriptionSQL', @description_sql);
insert into ofproperty(name, propValue) values('jdbcGroupProvider.loadMembersSQL', @load_members_sql);
insert into ofproperty(name, propValue) values('jdbcGroupProvider.loadAdminsSQL', @load_admin_sql);

update ofproperty set propValue ='org.jivesoftware.openfire.auth.JDBCAuthProvider' where name='provider.auth.className';

update ofproperty set propValue='org.jivesoftware.openfire.user.JDBCUserProvider' where name='provider.user.className';

update ofproperty set propValue='org.jivesoftware.openfire.group.JDBCGroupProvider' where name='provider.group.className';
rollback transaction
```



集成用户组以后，在聊天客户端中可以获取相关的组，但是不可直接针对群组发消息，如果要对群组发消息得先获取群组成员列表，然后循环分别发送。其实Openfire服务器可以帮我们完成消息的分发，只需针对群组id发送一条消息，剩下的任务完全可以交代给Openfire服务器完成，在系统的管理界面中安装其官方提供的广播插件：broadcast.jar，这个插件也提供了比较完善的文档参考，可在Openfire官方网站上查看，执行以下sql语句：

```sql
begin transaction
--broadcase service name
insert into ofproperty(name, propValue) values('plugin.broadcast.serviceName', 'broadcast');
--是否允许所有用户拥有给组发消息的权限,true为允许，false管理员有发送权限
insert into ofproperty(name, propValue) values('plugin.broadcast.disableGroupPermissions', 'true');
--是否允许组内用户发送广播消息
insert into ofproperty(name, propValue) values('plugin.broadcast.groupMembersAllowed', 'true');
--
--insert into ofproperty(name, propValue) values('plugin.broadcast.allowedUsers', '');
--设置是否给离线的用户发消息，true发送
insert into ofproperty(name, propValue) values('plugin.broadcast.all2offline', 'true');
--设置消息的附加前缀
insert into ofproperty(name, propValue) values('plugin.broadcast.messagePrefix', '');
rollback transaction
```



至此集成完成。