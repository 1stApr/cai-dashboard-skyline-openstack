# Giới thiệu chung
- Tài liệu này hướng dẫn cài đặt Skyline Dashboard (được tối ưu cho UI và UE) cho OpenSack.
- Môi trường OpenStack sử dụng trong tài liệu này được triển khai bằng Kolla-Ansible, CSDL là MariaDB.
# Trước khi bắt đầu 
- Cần có một môi trường OpenStack chạy các dịch vụ cơ bản ([Hướng dẫn cài OpenStack Sử dụng Kolla-Ansible](https://github.com/1stApr/cai-openstack-tren-virtualbox))
- Có một máy chủ Linux chạy Docker

# Cài Skyline Dashboard
## Tạo và chỉnh sửa file skyline.yaml

```shell
mkdir /etc/skyline/
curl https://raw.githubusercontent.com/1stApr/cai-dashboard-skyline-openstack/master/skyline.yaml -o /etc/skyline/skyline.yaml
```
```
vi /etc/skyline/skyline.yaml
```
**[Mẫu file skyline.yaml](skyline.yaml)**

Các thông số cần chỉnh sửa:
- database_url
- keystone_url
- default_region
- interface_type
- system_project_domain
- system_project
- system_user_domain
- system_user_name
- system_user_password

ví dụ:
```shell
database_url: mysql://root:PASSDB@localhost:3306/skyline
keystone_url: http://localhost:5000
default_region: RegionOne
interface_type: public
system_project_domain: Default
system_project: admin
system_user_domain: default
system_user_name: admin
system_user_password: USER_PASS
```

## Deployment với MariaDB

### Kết nối CSDL của OpenStack

```shell
docker exec -u0 -it mariadb bash
```

```shell
mysql -u root -p
```

### Tạo Skyline DB
```mysql
CREATE DATABASE IF NOT EXISTS skyline DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

### Cấp quyền truy cập vào CSDL

```mysql
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'localhost' IDENTIFIED BY 'SKYLINE_DBPASS';
```

```mysql
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'%'  IDENTIFIED BY 'SKYLINE_DBPASS';
```

```shell
exit 
exit 
```

### Tạo skyline user

Source the admin credentials
```shell
source admin-openrc.sh
```

Tạo skyline user
```shell
openstack user create --domain default --password-prompt skyline
```

Thêm admin role cho skyline user
```shell
openstack role add --project service --user skyline admin
```

## Chạy Skyline container
### Chạy skyline_bootstrap container để bootstrap

```shell
docker run -d --name skyline_bootstrap -e KOLLA_BOOTSTRAP="" -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml --net=host quay.io/tuanta52/skyline-openstack:latest
```

### Kiểm tra bootstrap có bình thường hay không (`exit 0` là bình thường)
```shell
docker logs skyline_bootstrap
```
###  Chạy skyline service sau khi bootstrap hoàn thành

```shell
docker rm -f skyline_bootstrap
```
```shell
docker run -d --name skyline --restart=always -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml --net=host quay.io/tuanta52/skyline-openstack:latest
```

## Truy cập vào Dashboard

<img src="/src/image_dashboard.png" width="900"> 

