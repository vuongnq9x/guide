
> Hướng dẫn này thực hiện trên ubuntu 24.04, php8.1 (Phiên bản ubuntu và php khác có thể có chút khác biệt)

### Cài đặt wget
```bash
sudo apt install -y wget
```

### Tạo thư mục chứa bộ SDK của ORACLE và tải các thư viện cần thiết
```bash
sudo mkdir -p /opt/oracle

cd /opt/oracle
# Tải SDK
sudo wget https://download.oracle.com/otn_software/linux/instantclient/2340000/instantclient-sdk-linux.x64-23.4.0.24.05.zip
sudo wget https://download.oracle.com/otn_software/linux/instantclient/2340000/instantclient-sqlplus-linux.x64-23.4.0.24.05.zip
sudo wget https://download.oracle.com/otn_software/linux/instantclient/2340000/instantclient-basic-linux.x64-23.4.0.24.05.zip

# Giải nén vào thư mục hiện tại sẽ được 1 thu mục instantclient_23_4
sudo apt unzip -A instantclient-sdk-linux.x64-23.4.0.24.05.zip
sudo apt unzip -A instantclient-sqlplus-linux.x64-23.4.0.24.05.zip
sudo apt unzip -A instantclient-basic-linux.x64-23.4.0.24.05.zip

echo "hello" | sudo tee /etc/ld.so.conf.d/oracle-insantclient.conf
sudo ldconfig

# Xóa các file zip nếu muốn
rm -rf *.zip
```

### Tải mã nguồn php tương ứng với phiên bản php hiện tại từ php.net (8.1.29)
```bash
cd /opt/oracle
sudo wget https://www.php.net/distributions/php-8.1.29.tar.gz
sudo tar -xvzf php-8.1.29.tar.gz
```
### Biên dịch và cài đặt pdo_oci 
```bash
cd /opt/oracle/php-8.1.29/ext/pdo_oci
sudo apt install -y php8.1-dev
sudo phpize8.1 .
sudo ./configure --with-pdo_oci=instantclient,/opt/oracle/instantclient_23_4
sudo make
sudo make install
```

### Biên dịch và cài đặt oci8 
```bash
cd /opt/oracle/php-8.1.29/ext/oci8
sudo phpize8.1 .
sudo ./configure --with-oci=instantclient,/opt/oracle/instantclient_23_4
sudo make
sudo make install
```

### Xử lí lỗi libaio1
```bash
# Với ubuntu 24.04 thì libaio1 đã được thay thế bằng libaio1t64, tuy nhiên 
# phía SDK chưa cập nhật nên sẽ cần thêm bước link thư viện này
sudo apt install -y libaio1t64
sudo ln -s /usr/lib/x86_64-linux-gnu/libaio.so.1t64 /usr/lib/x86_64-linux-gnu/libaio.so.1
```

### Thêm extension vào php.ini
```bash
echo 'extension=pdo_oci.so' | sudo tee -a /etc/php/8.1/fpm/php.ini
echo 'extension=oci8.so' | sudo tee -a /etc/php/8.1/fpm/php.ini
```

### Xác nhận việc cài đặt thành công
> Khởi động lại PHP
```bash
sudo service php8.1-fpm restart
```

> Kiểm tra vesion
```bash
php -v
```
Output:
```bash
 PHP 8.1.29 (cli) (built: Jun  6 2024 16:53:54) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.29, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.29, Copyright (c), by Zend Technologies
```

> Kiểm tra extension
```bash
php -m
```
Output:
```
...
oci8
...
PDO_OCI
...
```
