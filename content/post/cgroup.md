---
title: "Cgroup"

date: 2019-01-06T11:07:09Z

categories:
# - category
# - subcategory
- Docker
tags:
# - tag1
# - tag2
- Cgroup

keywords:
# Define keywords for search engines. you can also define global keywords 
# in Hugo configuration file.
# - tech

thumbnailImage: /img/docker.png
# Image displayed in index view.

thumbnailImagePosition : left
# Display thumbnail image at the right of title 
# in index pages (right, left or bottom). thumbnailImagePosition 
# overwrite the setting thumbnailImagePosition in the theme configuration file.

coverImage:
# Image displayed in full size at the top of your post in post view. If thumbnail image is not configured, cover image is also used as thumbnail image.

coverSize:
# partial: cover image take a part of the screen height (60%), full: cover image take the entire screen height.

coverCaption:
# Add a caption under the cover image
    
gallery:
# - image 1
# - image 2
comments: true
summary: Cgroup tản mạn
# Custom excerpt text to show on the homepage.
---

Chào các bạn, vẫn là tôi - MinhKMA. Tôi đã chạy docker được năm rồi nhưng mới chỉ dừng lại ở docker-compose. Lần này quay lại với nó với mục đích tiến xa hơn với công nghệ container này.

Trong phần này chúng ta sẽ xem xét các Control Groups (cgroups), cung cấp một cơ chế để dễ dàng quản lý và giám sát tài nguyên hệ thống, bằng cách phân vùng những thứ như cpu time, system memory, disk và network bandwidth thành các nhóm, sau đó phân công nhiệm vụ cho các nhóm đó .

Hãy để tôi thử và giải thích Control Groups là gì và chúng cho phép bạn làm gì. Chúng ta hãy bắt đầu bằng một ví dụ:

```
Bạn có một resources intensive application trên máy chủ
```

Trong việc chia sẻ tài nguyên giữa tất cả các tiến trình trên một hệ thống, Linux is great. Nhưng với trường hợp bạn muốn **To allocate or guarantee, a greater amount to a specific application, or a set of applications** thì Cgroup sẽ giúp ích bạn. (Phần này xin phép mình không dịch nó :D)

Ví dụ: Giả sử  Tôi muốn gán hoặc cô lập tài nguyên cho ứng dụng, hãy tạo hai nhóm:

- Nhóm #1 sẽ dành cho hệ điều hành 
- Nhóm #2 sẽ dành cho ứng dụng của tôi

Sau đó chúng tôi có thể define thông tin về tài nguyên cho từng nhóm.

<img src="https://i.imgur.com/cAMm6QT.gif">

Hãy tập trung vào Nhóm #2 một lát. Tôi muốn quản lý cpu, memory, disk và network bandwidth cho ứng dụng nên tôi đã tạo ra một nhóm và giới hạn tài nguyên cho nhóm vừa tạo. Vì thế nhưng ứng dụng thuộc nhóm này không thể vượt quá tài nguyên mà tôi đã gán cho nhóm trước đó. 

<img src="https://i.imgur.com/v6T42tT.gif">

Như vậy bạn có thể thấy, các ứng dụng trong nhóm #2 không thể vượt quá được:

- 80% CPU
- 10GB memory 
- 80% of disk reads and writes
- 80% of our network bandwidth

Khi nhóm được tạo ra, bạn chỉ cần gán PIDs của các ứng dụng mình muốn và các ứng dụng sẽ tự động điều chỉ mà không cần phải khởi động lại hệ thống. 

Thay vào đó chúng ta có hệ điều hành và có thể có nhiều nhóm: 

<img src='https://i.imgur.com/RWlYlg5.png'>

#### Nào, để dễ hình dung hơn tôi sẽ làm một ví dụ cụ thể:


- Cài đặt libcgroup

    ```
    yum install libcgroup libcgroup-tools -y
    systemctl start cgconfig
    systemctl enable cgconfig
    ```

    <img src="https://i.imgur.com/PokhIt1.png">

- Tạo demo1 trong blkio control group

    ```
    [root@pihole ~]# mkdir /sys/fs/cgroup/blkio/demo1
    [root@pihole demo1]# cd /sys/fs/cgroup/blkio/demo1
    [root@pihole demo1]# lscgroup
    [root@pihole demo1]# lscgroup | grep demo1
    blkio:/demo1
    [root@pihole demo1]# 
    ```

- Thực hiện việc đọc ghi dữ liệu thông thường

    ```
    [root@pihole ~]# dd if=/dev/zero of=file-abc bs=1M count=2000
    2000+0 records in
    2000+0 records out
    2097152000 bytes (2,1 GB) copied, 26,1617 s, 80,2 MB/s
    [root@pihole ~]# dd if=/dev/zero of=file-xyz bs=1M count=2000
    2000+0 records in
    2000+0 records out
    2097152000 bytes (2,1 GB) copied, 23,308 s, 90,0 MB/s
    [root@pihole ~]# free -m
                total        used        free      shared  buff/cache   available
    Mem:           3790         195         135           8        3459        3231
    Swap:          2047           1        2046
    [root@pihole ~]# echo 3 > /proc/sys/vm/drop_caches
    [root@pihole ~]# free -m
                total        used        free      shared  buff/cache   available
    Mem:           3790         186        2913           8         690        3349
    Swap:          2047           1        2046
    [root@pihole ~]# 
    ```

- Giới hạn rw trong group demo1

    ```
    [root@pihole demo1]# ls -l /dev/sda*
    brw-rw---- 1 root disk 8, 0 Jan  4 16:08 /dev/sda
    brw-rw---- 1 root disk 8, 1 Jan  4 16:08 /dev/sda1
    brw-rw---- 1 root disk 8, 2 Jan  4 16:08 /dev/sda2
    [root@pihole demo1]# echo "8:0 5242880" > blkio.throttle.read_bps_device
    [root@pihole demo1]# echo 3 > /proc/sys/vm/drop_caches
    [root@pihole demo1]# free -m
                total        used        free      shared  buff/cache   available
    Mem:           3790         175        3511           8         103        3429
    Swap:          2047           1        2046
    [root@pihole demo1]# 
    ```

    **Ở đây mình đã giới hạn read 5MB cho demo1**

- Kiểm tra: 

    ```
    [root@pihole ~]# cgexec -g blkio:demo1 dd if=file-abc of=/dev/null
    2363041+0 records in
    2363040+0 records out
    1209876480 bytes (1,2 GB) copied, 246,069 s, 4,9 MB/s
    [root@pihole ~]# 
    ```
- Kết quả đúng như gì mong đợi phải không? `4,9 MB/s` so với ban đầu là `90,0 MB/s`

### Ram. To be continue >>>
