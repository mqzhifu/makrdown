# gitlab cicd

wget https://mirrors.tuna.tsinghua.edu.cn/gitlab\-ce/yum/el7/gitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm

rpm \-Uvh gitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm

```

GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

```

external\_url 'https://192.168.31.120:5566'

gitlab\-ctl reconfigure

```
Warnings:
LD_LIBRARY_PATH was found in the env variables, this may cause issues with linking against the included libraries.

Notes:
Default admin account has been configured with following details:
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

NOTE: Because these credentials might be present in your log files in plain text, it is highly recommended to reset the password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.
```

gitlab\-ctl restart

登陆，添加用户

创建项目

增加.readme .cicd.yaml

//方法2\-官网

```
sudo yum install -y curl policycoreutils-python openssh-server perl

sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld

```

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

错误：

> Job for postfix.service failed because the control process exited with error code. See "systemctl status postfix.service" and "journalctl \-xe" for details.

vim /etc/postfix/main.conf

> curl https://packages.gitlab.com/install/repositories/gitlab/gitlab\-ee/script.rpm.sh | sudo bash

sudo EXTERNAL\_URL="http://gitlab.xlsyfx.cn:6677" yum install \-y gitlab\-ee

wget%20https%3A%2F%2Fmirrors.tuna.tsinghua.edu.cn%2Fgitlab\-ce%2Fyum%2Fel7%2Fgitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm%0A%0Arpm%20\-Uvh%20gitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm%0A%0A%60%60%60%0A%0AGitLab%20was%20unable%20to%20detect%20a%20valid%20hostname%20for%20your%20instance.%0APlease%20configure%20a%20URL%20for%20your%20GitLab%20instance%20by%20setting%20%60external\_url%60%0Aconfiguration%20in%20%2Fetc%2Fgitlab%2Fgitlab.rb%20file.%0AThen%2C%20you%20can%20start%20your%20GitLab%20instance%20by%20running%20the%20following%20command%3A%0A%20%20sudo%20gitlab\-ctl%20reconfigure%0A%0A%60%60%60%0A%0A%0Aexternal\_url%20'https%3A%2F%2F192.168.31.120%3A5566'%0Agitlab\-ctl%20reconfigure%0A%60%60%60%0AWarnings%3A%0ALD\_LIBRARY\_PATH%20was%20found%20in%20the%20env%20variables%2C%20this%20may%20cause%20issues%20with%20linking%20against%20the%20included%20libraries.%0A%0A%0ANotes%3A%0ADefault%20admin%20account%20has%20been%20configured%20with%20following%20details%3A%0AUsername%3A%20root%0APassword%3A%20You%20didn't%20opt\-in%20to%20print%20initial%20root%20password%20to%20STDOUT.%0APassword%20stored%20to%20%2Fetc%2Fgitlab%2Finitial\_root\_password.%20This%20file%20will%20be%20cleaned%20up%20in%20first%20reconfigure%20run%20after%2024%20hours.%0A%0ANOTE%3A%20Because%20these%20credentials%20might%20be%20present%20in%20your%20log%20files%20in%20plain%20text%2C%20it%20is%20highly%20recommended%20to%20reset%20the%20password%20following%20https%3A%2F%2Fdocs.gitlab.com%2Fee%2Fsecurity%2Freset\_user\_password.html%23reset\-your\-root\-password.%0A%60%60%60%0A%0Agitlab\-ctl%20restart%0A%0A%E7%99%BB%E9%99%86%EF%BC%8C%E6%B7%BB%E5%8A%A0%E7%94%A8%E6%88%B7%0A%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE%0A%E5%A2%9E%E5%8A%A0.readme%20.cicd.yaml%0A%0A%2F%2F%E6%96%B9%E6%B3%952\-%E5%AE%98%E7%BD%91%0A%60%60%60%0Asudo%20yum%20install%20\-y%20curl%20policycoreutils\-python%20openssh\-server%20perl%0A%0Asudo%20systemctl%20enable%20sshd%0Asudo%20systemctl%20start%20sshd%0A%0Asudo%20firewall\-cmd%20\-\-permanent%20\-\-add\-service%3Dhttp%0Asudo%20firewall\-cmd%20\-\-permanent%20\-\-add\-service%3Dhttps%0Asudo%20systemctl%20reload%20firewalld%0A%0A%60%60%60%0A%0A%60%60%60%0Asudo%20yum%20install%20postfix%0Asudo%20systemctl%20enable%20postfix%0Asudo%20systemctl%20start%20postfix%0A%60%60%60%0A%0A%0A%E9%94%99%E8%AF%AF%EF%BC%9A%0A%3EJob%20for%20postfix.service%20failed%20because%20the%20control%20process%20exited%20with%20error%20code.%20See%20%22systemctl%20status%20postfix.service%22%20and%20%22journalctl%20\-xe%22%20for%20details.%0A%0Avim%20%2Fetc%2Fpostfix%2Fmain.conf%0A%0A%3Ecurl%20https%3A%2F%2Fpackages.gitlab.com%2Finstall%2Frepositories%2Fgitlab%2Fgitlab\-ee%2Fscript.rpm.sh%20%7C%20sudo%20bash%0A%0AEXTERNAL\_URL%3D%22http%3A%2F%2Fgitlab.xlsyfx.cn%3A6677%22%20yum%20install%20\-y%20gitlab\-ee
