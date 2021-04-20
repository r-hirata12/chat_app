FROM centos:8

# hostとdockerの日付合わせ
RUN /bin/cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime

# CentOS 8 ではdnfコマンドがyumに代行する。
RUN dnf update -y

# apacheのinstall
RUN dnf -y install httpd httpd-tools httpd-devel httpd-manual

# epel リポジトリをinstall
RUN dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

# ./apache.confをコンテナにコピー（apache.confにphpのコンテナを指定して、このコンテナとphpのコンテナを接続させる）
COPY ./apache/conf/apache.conf /etc/httpd/conf.d/apache.conf