---
title: "DNS hoạt động ra sao?"

date: 2018-10-02T21:29:33+07:00

categories:
- DNS
# - category
# - subcategory

tags:
- DNS
# - tag1
# - tag2

keywords:
# Define keywords for search engines. you can also define global keywords 
# in Hugo configuration file.
# - tech
thumbnailImage: /img/5.jpg
# thumbnailImage: //example.com/image.jpg
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
summary: Tổng quan về DNS.
# Custom excerpt text to show on the homepage.
---

# Lời mở đầu 

Tại thời điểm bây giờ các từ khóa như Cloud, docker, CICD, Machine Learning, Big Data ... được nhắc đến nhiều trong thời đại công nghệ 4.0 này thì tôi lại đi viết một bài tìm hiểu về DNS. Trước đây tôi nghĩ DNS là những kiến thức căn bản mà hầu như ai cũng biết và chẳng có gì để viết về nó, nhưng gần đây tôi đã dành thời gian của mình để hệ thống lại những kiến thức căn bản. Để viết được một bài văn hay thì phải học cách sử dụng các câu đơn linh hoạt, để xây được căn nhà vững chắc thì móng nhà phải chuẩn bị thật tốt. Hy vọng bài viết này của tôi sẽ giúp các bạn có một cái nhìn chi tiết và trực quan hơn về DNS. 

# Tổng quan về DNS 

## Mục lục 

- [I. DNS là gì? ](#1)
- [II. Chức năng](#2)
- [III. Cấu trúc của DNS](#3)    
    + [1. Cấu trúc cơ sở dữ liệu](#4)
    + [2. Các bản ghi](#5)
- [IV. Workflow](#6)
- [V. Sử dụng tcpdump và wireshark để phân tích](#7)
- [VI. Tài liệu tham khảo](#8)

<a name="1"></a>
## I.DNS là gì? 

- DNS là từ viết tắt của Domain Name System, là Hệ thống phân giải tên được phát minh vào năm 1984 cho Internet, chỉ một hệ thống cho phép thiết lập tương ứng giữa địa chỉ IP và tên miền. Hệ thống tên miền (DNS) là một hệ thống đặt tên theo thứ tự cho máy vi tính, dịch vụ, hoặc bất kỳ nguồn lực tham gia vào Internet. Nó liên kết nhiều thông tin đa dạng với tên miền được gán cho những người tham gia. Quan trọng nhất là, nó chuyển tên miền có ý nghĩa cho con người vào số định danh (nhị phân), liên kết với các trang thiết bị mạng cho các mục đích định vị và địa chỉ hóa các thiết bị khắp thế giới. 

- Nó phục vụ như một “Danh bạ điện thoại” để tìm trên Internet bằng cách dịch tên máy chủ máy tính thành địa chỉ IP 

Ví dụ, www.example.com dịch thành 208.77.188.166.

- Hệ thống tên miền giúp cho nó có thể chỉ định tên miền cho các nhóm người sử dụng Internet trong một cách có ý nghĩa, độc lập với mỗi địa điểm của người sử dụng. Bởi vì điều này, World-Wide Web (WWW) siêu liên kết và trao đổi thông tin trên Internet có thể duy trì ổn định và cố định ngay cả khi định tuyến dòng Internet thay đổi hoặc những người tham gia sử dụng một thiết bị di động. Tên miền internet dễ nhớ hơn các địa chỉ IP như là 208.77.188.166 (IPv4) hoặc 2001: db8: 1f70:: 999: de8: 7648:6 e8 (IPv6).

- Mọi người tận dụng lợi thế này khi họ thuật lại có nghĩa các URL và địa chỉ email mà không cần phải biết làm thế nào các máy sẽ thực sự tìm ra chúng.


### Phân loại các Domain Name Server.

- Tên miền riêng (Primary Name Server) : Mỗi một máy chủ tên miền có mọt tên miền riêng. Tên miền riêng này được đăng ký trên internet.

- Tên miền dự phòng - tên miền thứ hai (Secondary Name Server) : Đây là một DNS server được sử dụng để thay thế cho Primary server DNS server bằng cách sao lưu lại tất cả bản ghi dữ liệu trên Primary Name server và nếu Primary Name server bị gián đoạn thì nó sẽ đảm nhận việc phần giải ánh xạ tên miền và địa chỉ IP.

- Caching Name Server : Đây là một server đảm nhiệm việc lưu tất cả những tên miền, địa chỉ IP, đã được phân giải và ánh xạ thành công. Nó được dùng trong các trường hợp sau :

    + Làm tăng tốc độ phân giải bằng cách sử dụng cache
    + Làm giảm bớt gánh nặng phân giải tên máy cho các DNS server
    + Giảm lưu lượng tham gia vào mạng và giảm độ trễ trên mạng (rất quang trọng).

<a name="2"></a>
## II. Chức năng của DNS

Mỗi Website có một tên (là tên miền hay đường dẫn URL:Universal Resource Locator) và một địa chỉ IP. Địa chỉ IP gồm 4 nhóm số cách nhau bằng dấu chấm(Ipv4). Khi mở một trình duyệt Web và nhập tên website, trình duyệt sẽ đến thẳng website mà không cần phải thông qua việc nhập địa chỉ IP của trang web. Quá trình "dịch" tên miền thành địa chỉ IP để cho trình duyệt hiểu và truy cập được vào website là công việc của một DNS server. Các DNS trợ giúp qua lại với nhau để dịch địa chỉ "IP" thành "tên" và ngược lại. Người sử dụng chỉ cần nhớ "tên", không cần phải nhớ địa chỉ IP (địa chỉ IP là những con số rất khó nhớ)

<a name="3"></a>
## III. Cấu trúc của DNS 

<a name="4"></a>
### 1. Cấu trúc cơ sở dữ liệu.

- Cơ sở dữ liệu của hệ thống DNS là hệ thống cơ sở dữ liệu phân tán và phân cấp hình cây. Với `.Root server` là đỉnh của cây và sau đó các miền (domain) được phân nhánh dần xuống dưới và phân quyền quản lý.

- Khi một máy khách (client) truy vấn một tên miền nó sẽ đi lần lượt từ root phân cấp xuống dưới để đến DNS quản lý domain cần truy vấn.

#### Zone 

- Hệ thống tên miền(DNS) cho phép phân chia tên miền để quản lý và nó chia hệ thống tên miền thành zone và trong zone quản lý tên miền được phân chia đó.

- Các Zone chứa thông tin vê miền cấp thấp hơn, có khả năng chia thành các zone cấp thấp hơn và phân quyền cho các DNS server khác quản lý.

- Ví dụ : Zone “.vn” thì do DNS server quản lý zone “.vn” chứa thông tin về các bản ghi có đuôi là “.vn” và có khả năng chuyển quyền quản lý (delegate) các zone cấp thấp hơn cho các DNS khác quản lý như “.fpt.vn” là vùng (zone) do fpt quản lý.

Hệ thống cơ sở dữ liệu của DNS là hệ thống dữ liệu phân tán hình cây như cấu trúc đó là cấu trúc logic trên mạng Internet

<img src="https://i.imgur.com/LaiHTFV.png">


<a name="5"></a>
### 2.Các bản ghi thường có trong cơ sở dữ liệu của DNS server

#### 2.1 Bản ghi SOA (Start of Authority )

- Bản ghi này xác định máy chủ DNS có thẩm quyền cung cấp thông tin về tên miền xác định trên DNS.

- Trong mỗi tập tin CSDL phải có 1 và chỉ 1 record SOA. Bảng ghi SOA này chỉ ra rằng Primary Name Server là nơi cung cấp thông tin tin cậy từ dữ liệu có trong zone.

- Cú pháp:

```
[tên-miền] IN SOA [tên-DNS-Server] [địa-chỉ-email] (

Serial number;

Refresh number;

Retry number;

Experi number;

Time-to-line number)
```

- Ví dụ: 

```
nhanhoa.com. IN SOA ns.nhanhoa.com. root.nhanhoa.com. (

1 ; serial

10800 ; refresh after 3 hours

3600 ; retry after 1 hours

604800 ; expire after 1 week

86400 ) ; minimum TTL of 1 day
```

Giải thích ý nghĩa ví dụ trên :

- Tên Domain : nhanhoa.com. phải ở vị trí cột đầu tiên và kết thúc bằng dấu chấm (.).
- IN là Internet
- ns.nhanhoa.com. là tên FQDN của Primary Name Server của dữ liệu này.
- root.nhanhoa.com. là địa chỉ email của người phụ trách dữ liệu này. Lưu ý là địa chỉ email thay thế dấu @ bằng dấu chấm sau root.
- Dấu ( ) cho phép ta mở rộng ra viết thành nhiều dòng, tất cả các tham số trong dấu ( ) được dùng cho các Secondary Name Server.

Các thành phần bên trong cú pháp của record SOA :

- Serial : áp dụng cho mọi dữ liệu trong zone và là 1 số nguyên. Trong ví dụ, giá trị này là 1 nhưng thông thường người ta sẽ sử dụng theo định dạng thời gian như 2007092001. Định dạng này theo kiểu yyyymmddnn, trong đó nn là số lần sửa đổi dữ liệu zone trong ngày. Bất kể theo định dạng nào thì luôn luôn phải tăng số này lên mỗi lần sửa đổi dữ liệu zone. Khi Secondary Name Server liên lạc với Primary Name Server thì trước tiên nó sẽ hỏi số serial này. Nếu số serial của máy Secondary nhỏ hơn số serial của máy Primary tức là dữ liệu trên Secondary đã cũ và sau đó máy Secondary sẽ sao chép dữ liệu mới từ máy Primary thay cho dữ liệu đang có.
- Refresh : chỉ ra khoản thời gian máy Secondary kiểm tra dữ liệu zone trên máy Primary để cập nhật nếu cần. Trong ví dụ trên thì cứ mổi 3 giờ máy chủ Secondary sẽ liên lạc với máy chủ Primary để cập nhật nếu có. Giá trị này thay đổi theo tần suất thay đổi dữ liệu trong zone.
- Retry : nếu máy Secondary không kết nối được với máy Primary theo thời hạn mô tả trong refresh (ví dụ trường hợp máy Primary shutdown máy vào lúc đó) thì máy Secondary sẽ tìm cách kết nối lại với máy Primary theo chu kỳ thời gian được xác định trong retry. Thông thường giá trị này nhỏ hơn giá trị refresh
- Expire : nếu sau khoản thời gian này mà máy Secondary không cập nhật được thông tin mới trên máy Primay thì giá trị của zone này trên máy Secondary sẽ bị hết hạn. Nếu bị expire thì Secondary sẽ không trả lời bất cứ 1 truy vấn nào về zone này. Giá trị expire này phải lớn hơn giá trị refresh và giá trị retry.
- TTL : giá trị này áp dụng cho mọi record trong zone và được đính kèm trong thông tin trả lời 1 truy vấn. Mục đích của nó là chỉ ra thời gian mà các máy DNS Server khác cache lại thông tin trả lời. Giúp giảm lưu lượng truy vấn DNS trên mạng.


#### 2.1 Bản ghi NS 

- Bản ghi NS dùng để khai báo máy chủ tên miền cho một tên miền. Nó cho biết các thông tin về tên miền quản lý, do đó yêu cầu có tối thiểu hai bản ghi NS cho mỗi tên miền.
- Cú pháp của bản ghi NS:

    ``<tên miền> IN NS <tên của máy chủ tên miền>``

- Ví dụ:

    ```
    nhanhoa.com IN NS dns1.nhanhoa.com
    nhanhoa.com IN NS dns2.nhanhoa.com
    ```

Với khai báo trên, tên miền nhanhoa.com sẽ do máy chủ tên miền có tên dns.nhanhoa.com quản lý. Điều này có nghĩa, các bản ghi như A, CNAME, MX … của tên miền cấp dưới của nó sẽ được khai báo trên máy chủ dns1.nhanhoa.com. và dns2.nhanhoa.com.

#### 2.3 Bản ghi kiểu A 

- Bản ghi kiểu A được dùng để khai báo ánh xạ giữa tên của một máy tính trên mạng và địa chỉ IP của một máy tính trên mạng.
- Bản ghi kiểu A có cú pháp như sau:

    ``Domain IN A <địa chỉ IP của máy>``

- Ví dụ :

    `nhanhoa.com IN A 10.10.10.100`

    Theo ví dụ trên, tên miền nhanhoa.com được khai với bản ghi kiểu A trỏ đến địa chỉ 10.10.10.100 sẽ là tên của máy tính này.

- Một tên miền có thể được khai nhiều bản ghi kiểu A khác nhau để trỏ đến các địa chỉ IP khác nhau. Như vậy có thể có nhiều máy tính có cùng tên trên mạng. Ngược lại một máy tính có một địa chỉ IP có thể có nhiều tên miền trỏ đến, tuy nhiên chỉ có duy nhất một tên miền được xác định là tên của máy, đó chính là tên miền được khai với bản ghi kiểu A trỏ đến địa chỉ của máy.

#### 2.4 Bản ghi kiểu AAAA

- Ánh xạ tên máy (hostname) vào địa chỉ IP version 6
- Cú pháp :
    
    ``[tên-máy-tính] IN AAAA [địa-chỉ-IPv6]``

- Ví dụ :

    ``Server IN AAAA 1243:123:456:7892:3:456ab``

#### 2.5 Bản ghi CNAME

- Bản ghi CNAME cho phép một máy tính có thể có nhiều tên. Nói cách khác bản ghi CNAME cho phép nhiều tên miền cùng trỏ đến một địa chỉ IP cho trước.

- Để có thể khai báo bản ghi CNAME , bắt buộc phải có bản ghi kiểu A để khai báo tên của máy. Tên miền được khai báo trong bản ghi kiểu A trỏ đến địa chỉ IP của máy được gọi là tên miền chính (canonical domain ). Các tên miền khác muốn trỏ đến máy tính này phải được khai báo là bí danh của tên máy (alias domain).

- Bản ghi CNAME có cú pháp như sau :

    ``alias-domain IN CNAME canonical domain``

- Ví dụ :

    ``www.nhanhoa.com IN CNAME nhanhoa.com``

Tên miền www.nhanhoa.com sẽ là tên bí danh của tên miền nhanhoa.com, hai tên miền www.nhanhoa.com sẽ cùng trỏ đến địa chỉ IP 10.10.10.100

#### 2.6 Bản ghi MX

- Bản ghi MX dùng để khai báo trạm chuyển tiếp thư điện tử của một tên miền.

- Ví dụ : Để các thư điện tử có cấu trúc user@nhanhoa.com được gửi đến trạm chuyển tiếp thư điện tử có tên mail.nhanhoa.com, trên cơ sở dữ liệu cần khai báo bản ghi MX như sau:

    ``nhanhoa.com IN MX 10 mail.nhanhoa.com``

- Các thông số được khai báo trong bản ghi MX nêu trên gồm có:

    + nhanhoa.com : là tên miền được khai báo để sử dụng như địa chỉ thư điện tử.
    + mail.nhanhoa.com: là tên của trạm chuyển tiếp thư điện tử, nó thực tế là tên của máy tính dùng làm máy trạm chuyển tiếp thư điện tử.
    + 10: Là giá tri ưu tiên, giá trị ưu tiên có thể là một số nguyên bất kì từ 1 đến 225, nếu giá trị ưu tiên này càng nhỏ thì trạm chuyển tiếp thư điện tử được khai báo sau đó sẽ là trạm chuyển tiếp thư điện tử được chuyển đến đầu tiên.

#### 2.7 Bản ghi PTR

- Hệ thống DNS không những thực hiện việc chuyển đổi từ tên miền sang địa chỉ IP mà còn thực hiện chuyển đổi địa chỉi IP mà còn thực hiện chuyển đổi địa chỉ IP sang tên miền.

- Bản ghi PTR cho phép thực hiện chuyển đổi địa chỉ IP sang tên miền. Cú pháp của bản ghi PTR:

    ``100.10.10.10.in-addr.arpa IN PTR www.nhanhoa.com``

<a name="6"></a>
### IV. Workflow 

<img src="https://i.imgur.com/OTJoDib.png">

1. Khi máy tính của bạn cần kết nối với máy chủ lưu trữ trên Internet (ví dụ: MyGreatName.com), bạn chỉ cần nhập Tên miền (ví dụ: MyGreatName.com) vào URL của trình duyệt. Sau đó, máy tính của bạn sẽ liên hệ với Name Servers được cấu hình hoặc mặc định (thường là ISP Name Server), yêu cầu địa chỉ IP của máy chủ lưu trữ (ví dụ MyGreatName.com).

2. Nếu ISP Name Server của bạn có thông tin về địa chỉ IP của máy chủ truy vấn, nó sẽ cho máy tính của bạn biết ngay 

3. Giả sử rằng ISP Name Server của bạn không có thông tin của MyGreatName.com. ISP Name Server của bạn sẽ hỏi DNS Root Name Server ngay lập tức Name Server có thông tin của MyGreatName.com.

    ISP Name Server của bạn có thể biết máy chủ Root Name như thế nào? Root Name Server nào cần hỏi?

    Trên thực tế tất cả các Name Server sẽ tải về và cài đặt một tệp tin từ máy chủ FTP của liên quốc tế. File được gọi là "named.cache" hoặc "named.root". File đó có địa chỉ IP của tất cả Root Name Servers.

    Đây là file named.cache năm 2005: 

    ```
    ; This file holds the information on root name servers 
    ; needed to initialize cache of Internet domain name 
    ; servers (e.g. reference this file in the 
    ; "cache  .  " configuration file of BIND domain 
    : name servers).
    ;
    ; This file is made available by InterNIC registration 
    ; services under anonymous FTP as
    ;     file             /domain/named.root
    ;     on server        FTP.RS.INTERNIC.NET
    ; -OR- under Gopher at RS.INTERNIC.NET
    ;     under menu     InterNIC Registration Services (NSI)
    ;        submenu     InterNIC Registration Archives
    ;     file           named.root
    ;
    ; last update:    Aug 22, 1997
    ; related version of root zone:   1997082200
    ;
    ;
    ; formerly NS.INTERNIC.NET
    ;
    .                     3600000 IN  NS  A.ROOT-SERVERS.NET.
    A.ROOT-SERVERS.NET.   3600000     A   198.41.0.4
    ;
    ; formerly NS1.ISI.EDU
    ;
    .                     3600000     NS  B.ROOT-SERVERS.NET.
    B.ROOT-SERVERS.NET.   3600000     A   128.9.0.107
    ;
    ; formerly C.PSI.NET
    ;
    .                     3600000     NS  C.ROOT-SERVERS.NET.
    C.ROOT-SERVERS.NET.   3600000     A   192.33.4.12
    ;
    ; formerly TERP.UMD.EDU
    ;
    .                     3600000     NS  D.ROOT-SERVERS.NET.
    D.ROOT-SERVERS.NET.   3600000     A   128.8.10.90
    ;
    ; formerly NS.NASA.GOV
    ;
    .                     3600000     NS  E.ROOT-SERVERS.NET.
    E.ROOT-SERVERS.NET.   3600000     A   192.203.230.10
    ;
    ; formerly NS.ISC.ORG
    ;
    .                     3600000     NS  F.ROOT-SERVERS.NET.
    F.ROOT-SERVERS.NET.   3600000     A   192.5.5.241
    ;
    ; formerly NS.NIC.DDN.MIL
    ;
    .                     3600000     NS  G.ROOT-SERVERS.NET.
    G.ROOT-SERVERS.NET.   3600000     A   192.112.36.4
    ;
    ; formerly AOS.ARL.ARMY.MIL
    ;
    .                     3600000     NS  H.ROOT-SERVERS.NET.
    H.ROOT-SERVERS.NET.   3600000     A   128.63.2.53
    ;
    ; formerly NIC.NORDU.NET
    ;
    .                     3600000     NS  I.ROOT-SERVERS.NET.
    I.ROOT-SERVERS.NET.   3600000     A   192.36.148.17
    ;
    ; temporarily housed at NSI (InterNIC)
    ;
    .                     3600000     NS  J.ROOT-SERVERS.NET.
    J.ROOT-SERVERS.NET.   3600000     A   198.41.0.10
    ;
    ; housed in LINX, operated by RIPE NCC
    ;
    .                     3600000     NS  K.ROOT-SERVERS.NET.
    K.ROOT-SERVERS.NET.   3600000     A   193.0.14.129
    ;
    ; temporarily housed at ISI (IANA)
    ;
    .                     3600000     NS  L.ROOT-SERVERS.NET.
    L.ROOT-SERVERS.NET.   3600000     A   198.32.64.12
    ;
    ; housed in Japan, operated by WIDE
    ;
    .                     3600000     NS  M.ROOT-SERVERS.NET.
    M.ROOT-SERVERS.NET.   3600000     A   202.12.27.33
    ; End of File   
    ```

- Từ tệp named.cache nói trên, chúng ta biết rằng có 13 Name Server Root trên internet và được phân phối trên tất cả mọi nơi trên toàn thế giới 

- Root Name Servers có tất cả thông tin của Autoritative Domain Name Servers cho top level domain names (VD: .com, .org, .net, .com.hk, etc ..)

4. Khi ISP Name Server của bạn không có thông tin địa chỉ IP của MyGreatName.com, nó sẽ kiểm tra tập tin named.cache và yêu cầu trợ giúp từ Root NameServer. Nếu Root NameServer bị lỗi hoặc không có phản hồi, Name Server ISP của bạn sẽ hỏi Root NameServer thứ 2.

5. Root Name Serve sẽ nói ch ISP Name Server của bạn authoritative Name Server của MyGreatName.com là 212.69.192.10 Primary Name Server) và 212.69.192.11 (Secondary Name Server). 

    Bây giờ, bạn nên biết rằng tại sao bạn cần phải gửi thông tin của hai Name Server khi đăng ký tên miền mới.

6. Bây giờ ISP Name Server của bạn đã có địa chỉ ip của Authoritative Name Server MyGreatName.com. ISP Name Server của bạn sẽ liên lạc với Authoritative Name Server của MyGreatName.com (212.69.192.10). Authoritative Name Server của MyGreatName.com sẽ kiểm tra và xác nhận thông tin của MyGreatName.com. Sau đó nó cho biết địa chỉ IP của MyGreatName.com (212.69.204.148) với ISP của bạn.

7. ISP Name Server của bạn bây giờ có địa chỉ IP của MyGreatName.com, nó sẽ cho máy tính của bạn ngay lập tức.

8. Khi máy tính của bạn nhận được địa chỉ IP của MyGreatName.com, máy tính của bạn có thể giao tiếp với MyGreatName.com.

<a name="7"></a>
## V. Sử dụng TCPdump và wireshark phân tích gói tin

**Như đã đề cập ở trên về cách thức hoạt động của DNS server, bây giờ mình sẽ sử dụng tcpdump để phân tích gói tin kiểm chứng lại**

### Mô hình:

- Client
    + IP : 10.10.10.174
    + Domain Name : influxdb.com
- DNS server
    + IP : 10.10.10.173
    + Doman Server: minhkma.com

### Cài đặt:

Bạn có thể tham khảo ở <a href='https://github.com/MinhKMA/DNS_note/blob/master/docs/intro_dns.md'>đây</a>.

### Phân tích:

***Khi bạn gõ https://dantri.com.vn/ vào thanh tìm kiếm trên google chrome thì tên miền `dantri.com.vn` được ánh xạ đến IP như thế nào?***

<img src='https://i.imgur.com/YSat6Fr.png'>

- Client query ipv4 và ipv6 của tên miền `dantri.com.vn` 

<img src='https://i.imgur.com/skB1AJX.png'>

- Trên DNS server local chưa có thông tin của tên miền `dantri.com.vn` nên nó sẽ tiếp tục query đến dns root để hỏi thông tin.

```
16:04:02.194744 IP minhkma.com.31357 > M.ROOT-SERVERS.NET.domain: 34763% [1au] NS? . (28)
16:04:02.194838 IP minhkma.com.43583 > M.ROOT-SERVERS.NET.domain: 65507% [1au] A? dantri.com.vn. (42)
16:04:02.195004 IP minhkma.com.21549 > M.ROOT-SERVERS.NET.domain: 54637% [1au] AAAA? dantri.com.vn. (42)

```
<img src='https://i.imgur.com/WDw1Mx1.png'>

- Ở đây root server có địa chỉ IP là `202.12.27.33`

- Root server trả về thông tin tên miền `dantri.com.vn` được quản lý bởi các máy chủ:

```

    Authoritative nameservers
        vn: type NS, class IN, ns g.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            Name Server: g.dns-servers.vn
        vn: type NS, class IN, ns a.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: a.dns-servers.vn
        vn: type NS, class IN, ns c.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: c.dns-servers.vn
        vn: type NS, class IN, ns f.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: f.dns-servers.vn
        vn: type NS, class IN, ns e.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: e.dns-servers.vn
        vn: type NS, class IN, ns d.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: d.dns-servers.vn
        vn: type NS, class IN, ns b.dns-servers.vn
            Name: vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Name Server: b.dns-servers.vn
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 24
            Key id: 0xba0b
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-1 (1)
            Digest: 0457d87762782ccc3d355d6aa87e28c37fb293c9
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 36
            Key id: 0xba0b
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-256 (2)
            Digest: 009f879f8dbab6a453eaccbff323354e72715e4ba8f77131...
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 24
            Key id: 0x0cbc
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-1 (1)
            Digest: 913c652b4006deac045b945f4ffcbc1065645421
        vn: type DS, class IN
            Name: vn
            Type: DS(Delegation Signer) (43)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 36
            Key id: 0x0cbc
            Algorithm: RSA/SHA-256 (8)
            Digest Type: SHA-256 (2)
            Digest: 5a58c19af266077ffe16c2668812796fa8193661af569dbc...
        vn: type RRSIG, class IN
            Name: vn
            Type: RRSIG (46)
            Class: IN (0x0001)
            Time to live: 86400
            Data length: 275
            Type Covered: DS(Delegation Signer) (43)
            Algorithm: RSA/SHA-256 (8)
            Labels: 1
            Original TTL: 86400 (1 day)
            Signature Expiration: Nov 15, 2018 12:00:00.000000000 +07
            Signature Inception: Nov  2, 2018 11:00:00.000000000 +07
            Key Tag: 2134
            Signer's name: <Root>
            Signature: f069a4db905f767b55f154d8775076a93d6952fe977e49b3...
    Additional records
        a.dns-servers.vn: type A, class IN, addr 194.0.1.18
            Name: a.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 194.0.1.18
        b.dns-servers.vn: type A, class IN, addr 203.119.73.105
            Name: b.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.73.105
        c.dns-servers.vn: type A, class IN, addr 203.119.38.105
            Name: c.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.38.105
        d.dns-servers.vn: type A, class IN, addr 203.119.44.105
            Name: d.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.44.105
        e.dns-servers.vn: type A, class IN, addr 203.119.60.105
            Name: e.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.60.105
        f.dns-servers.vn: type A, class IN, addr 203.119.68.105
            Name: f.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 203.119.68.105
        g.dns-servers.vn: type A, class IN, addr 204.61.216.115
            Name: g.dns-servers.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 4
            Address: 204.61.216.115
        a.dns-servers.vn: type AAAA, class IN, addr 2001:678:4::12
            Name: a.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:678:4::12
        b.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:1:2::105
            Name: b.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:1:2::105
        c.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:c000:7::105
            Name: c.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:c000:7::105
        f.dns-servers.vn: type AAAA, class IN, addr 2001:dc8:d000:2::105
            Name: f.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:dc8:d000:2::105
        g.dns-servers.vn: type AAAA, class IN, addr 2001:500:14:6115:ad::1
            Name: g.dns-servers.vn
            Type: AAAA (IPv6 Address) (28)
            Class: IN (0x0001)
            Time to live: 172800
            Data length: 16
            AAAA Address: 2001:500:14:6115:ad::1
        <Root>: type OPT
            Name: <Root>
            Type: OPT (41)
            UDP payload size: 4096
            Higher bits in extended RCODE: 0x00
            EDNS0 version: 0
            Z: 0x8000
                1... .... .... .... = DO bit: Accepts DNSSEC security RRs
                .000 0000 0000 0000 = Reserved: 0x0000
            Data length: 0
```

- DNS server local tiếp tục query tới máy chủ `d.dns-servers.vn.domain` có địa chỉ ip `203.119.44.105` 

<img src='https://i.imgur.com/LYgO4ja.png'>

- Máy chủ `d.dns-servers.vn.domain` trả về cho DNS server local thông tin authoritative nameservers

<img src='https://i.imgur.com/O3DknzW.png'>

- DNS server tìm kiếm địa chỉ IP của máy chủ `ns7.synerfy.vn` có địa chỉ IP là `123.30.51.247` từ máy chủ `ns2.vdconline.vn` có địa chỉ IP`123.30.176.243`

<img src='https://i.imgur.com/Y6qW1FD.png'>

- DNS server query `dantri.com.vn` với máy chủ `ns7.synerfy.vn` có địa chỉ IP là `123.30.51.247` vừa có. `ns7.synerfy.vn` trả lời lại địa chỉ ipv4 của `dantri.com.vn`

```
Frame 39: 258 bytes on wire (2064 bits), 258 bytes captured (2064 bits)
    Encapsulation type: Ethernet (1)
    Arrival Time: Nov  2, 2018 16:04:02.444771000 +07
    [Time shift for this packet: 0.000000000 seconds]
    Epoch Time: 1541149442.444771000 seconds
    [Time delta from previous captured frame: 0.000398000 seconds]
    [Time delta from previous displayed frame: 0.000398000 seconds]
    [Time since reference or first frame: 0.251647000 seconds]
    Frame Number: 39
    Frame Length: 258 bytes (2064 bits)
    Capture Length: 258 bytes (2064 bits)
    [Frame is marked: False]
    [Frame is ignored: False]
    [Protocols in frame: eth:ethertype:ip:udp:dns]
    [Coloring Rule Name: UDP]
    [Coloring Rule String: udp]
Ethernet II, Src: Vmware_22:dd:cf (00:0c:29:22:dd:cf), Dst: Vmware_b7:29:3c (00:50:56:b7:29:3c)
Internet Protocol Version 4, Src: 123.30.51.247, Dst: 10.10.10.173
User Datagram Protocol, Src Port: 53, Dst Port: 2764
Domain Name System (response)
    Transaction ID: 0x3d33
    Flags: 0x8400 Standard query response, No error
    Questions: 1
    Answer RRs: 4
    Authority RRs: 3
    Additional RRs: 4
    Queries
    Answers
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
        dantri.com.vn: type A, class IN, addr 14.225.10.14
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 14.225.10.14
        dantri.com.vn: type A, class IN, addr 123.30.151.72
            Name: dantri.com.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 300
            Data length: 4
            Address: 123.30.151.72
    Authoritative nameservers
        dantri.com.vn: type NS, class IN, ns ns7.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 14
            Name Server: ns7.synerfy.vn
        dantri.com.vn: type NS, class IN, ns ns8.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 6
            Name Server: ns8.synerfy.vn
        dantri.com.vn: type NS, class IN, ns ns6.synerfy.vn
            Name: dantri.com.vn
            Type: NS (authoritative Name Server) (2)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 6
            Name Server: ns6.synerfy.vn
    Additional records
        ns7.synerfy.vn: type A, class IN, addr 123.30.51.247
            Name: ns7.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 123.30.51.247
        ns8.synerfy.vn: type A, class IN, addr 210.245.87.125
            Name: ns8.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 210.245.87.125
        ns6.synerfy.vn: type A, class IN, addr 173.193.24.114
            Name: ns6.synerfy.vn
            Type: A (Host Address) (1)
            Class: IN (0x0001)
            Time to live: 3600
            Data length: 4
            Address: 173.193.24.114
        <Root>: type OPT
            Name: <Root>
            Type: OPT (41)
            UDP payload size: 1024
            Higher bits in extended RCODE: 0x00
            EDNS0 version: 0
            Z: 0x0000
                0... .... .... .... = DO bit: Cannot handle DNSSEC security RRs
                .000 0000 0000 0000 = Reserved: 0x0000
            Data length: 0
    [Request In: 33]
    [Time: 0.002556000 seconds]
```

- Cuối cùng DNS server local trả lại thông tin cho client thông tin tên miền `dantri.com.vn`

```
51	0.400378	10.10.10.173	10.10.10.174	DNS	167	Standard query response 0x65d5 A dantri.com.vn A 123.30.151.72 A 14.225.10.14 NS ns7.synerfy.vn NS ns6.synerfy.vn NS ns8.synerfy.vn
```

Qua bài viết này hy vọng sẽ giúp ích được các bạn khi tìm hiểu về  DNS. Bài viết của mình còn nhiều thiếu sót mong nhận được lời góp ý từ các bạn. 

<a name="8"></a>
## VI. Tài liệu tham khảo 

- https://www.verisign.com/en_US/website-presence/online/how-dns-works/index.xhtml
- http://www.mygreatname.com/how-dns-works/e-04-how-dns-works.htm
- https://quantrimang.com/tim-hieu-ve-dns-dns-lookup-la-gi-118286
- https://www.unixmen.com/setting-dns-server-centos-7/