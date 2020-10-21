# Các lỗi bảo mật trong Laravel và cách phòng tránh
Trong bài viết này bạn sẽ học được các kỹ thuật phòng tráng tấn công phổ biến như XSS, CSRF, SQL Injection, DDoS, replay attack...

# CSRF Protection
CSRF(Cross Site Request Forgery, còn gọi là Session riding) là kỹ thuật tấn công bằng cách sử dụng quyền chứng thực của người dùng đối với một website. CSRF là kỹ thuật tấn công vào người dùng, dựa vào đó hacker có thể thực thi những thao tác phải yêu cầu sự chứng thực. Hiểu một cách nôm na, đây là kỹ thuật tấn công dựa vào mượn quyền trái phép.

Ví dụ:
Gỉa sử hệ thống của bạn có 1 action xóa product nào đó chẳng hạn https://x.com/product/1/delete. Như vậy nếu có một người nào biết được link này thì họ sẽ hack được, họ có thể gửi mail cho bạn chứa link, hoặc nội dung là một hay nhiều thẻ img với src chính là url đó và mỗi hình có id khác nhau kiểu như <img height="0" width="0" src="https://x.com/product/1/delete"> , như vậy nếu bạn đọc cái nội dung đó và đang login vào hệ thống thì bạn đã vô tình xóa đi các product với id như trong các src của thẻ img trên. Hiện thì có nhiều cách họ gửi lắm, họ dùng các thẻ img, link, bgsound, background,...

Nói chung là cái nghiệp của chúng nó rồi, phải tìm cách phòng tránh thôi.

- Dùng đúng chuẩn restfull(GET, PUT, POST, DELETE) để thao tác với hệ thống.
- Sử dụng Capcha
- Kiểm tra Refer: Kiểm tra xem các câu lệnh http gửi đến hệ thống xuất phát từ đâu. Một ứng dụng web có thể hạn chế chỉ thực hiện các lệnh http gửi đến từ các trang đã được chứng thực.
- Kiểm tra IP
- Sử dụng token: Trong Laravel cung cấp cho bạn phòng tránh, bạn có thể thêm vào các Form:

```php
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```
Để biết thêm và cách hoạt động thì bạn tham khảo tài liệu này.
https://laravel.com/docs/8.x/csrf

# XSS (Cross Site Scripting)
XSS là một đoạn mã độc, để khái thác một lỗ hổng XSS, hacker sẽ chèn mã độc thông qua các đoạn script để thực thi chúng ở phía Client. Mục đích chính của cuộc tấn công này là ăn cắp dữ liệu nhận dạng của người dùng như: cookies, session tokens và các thông tin khác. 

Trong hầu hết các trường hợp, cuộc tấn công này đang được sử dụng để ăn cắp cookie của người khác. Như chúng ta biết, cookie giúp chúng tôi đăng nhập tự động. Do đó với cookie bị đánh cắp, chúng tôi có thể đăng nhập bằng các thông tin nhận dạng khác.

Có 3 dạng XSS:

1. Reflected XSS
Có nhiều hướng để khai thác thông qua lỗi Reflected XSS, một trong những cách được biết đến nhiều nhất là chiếm phiên làm việc (session) của người dùng, từ đó có thể truy cập được dữ liệu và chiếm được quyền của họ trên website.

2. Stored XSS
Khác với Reflected tấn công trực tiếp vào một số nạn nhân mà hacker nhắm đến, Stored XSS hướng đến nhiều nạn nhân hơn. Lỗi này xảy ra khi ứng dụng web không kiểm tra kỹ các dữ liệu đầu vào trước khi lưu vào database. Ví dụ như các form góp ý, các comment,... trên các trang web.
Ví dụ: 
<script>alert("Opps! Your website is Hacked.")</script>
Nếu trong 1 bình luận comment nào đó trên 1 website, mình hiển thị cái đoạn script này thì ai vào đọc cũng sẽ bị cái popup này hiển thị lên.

3. DOM Based XSS
DOM Based XSS là kỹ thuật khai thác XSS dựa trên việc thay đổi cấu trúc DOM của tài liệu, cụ thể là HTML.

CÁCH PHÒNG CHỐNG:
- Encode
Bất kỳ dữ liệu form nào được nhập vào cũng được encode và kiểm tra trước khi lưu.
Nếu bắt buộc cần hiển thị những thẻ html thì nên qui định những thẻ nào được nhập, những thẻ nào không được nhập cần mã hóa (thẻ script chẳng hạn).

Trong Laravel khi hiển thị dữ liệu trong blade, bạn nên dùng các hàm @{{}} để data được encode

- CSP (Content security Policy)
Với CSP thì có thể cấu hình trình duyệt chỉ chạy những mã javascript từ những domain được chỉ định sẵn mà thôi


# SQL Injection
SQL injection là một kỹ thuật cho phép những kẻ tấn công lợi dụng lỗ hổng của việc kiểm tra dữ liệu đầu vào trong các ứng dụng web và các thông báo lỗi của hệ quản trị cơ sở dữ liệu trả về để inject và thi hành các câu lệnh SQL bất hợp pháp.

Hậu quả của nó để lại thì rất quan ngại: lộ dữ liệu, chỉnh sửa dữ liệu và ảnh hưởng đến toàn bộ hệ thống bởi dữ liệu là trung tâm của cả ứng dụng.

Cách tấn công thì cũng đơn giản thôi, kiểu như thêm chuỗi "' OR drop tables users", hoặc với các request thì có thể "' OR 1=1" thì coi như đúng mọi trường hợp.

Cách phòng chống là:

- Không cộng chuỗi để tạo SQL: Sử dụng parameter thay vì cộng chuỗi. Nếu dữ liệu truyền vào không hợp pháp, SQL Engine sẽ tự động báo lỗi, ta không cần dùng code để check.
- Lọc dữ liệu từ người dùng: Không được tin user, lọc bất kì dữ liệu gì họ nhập vào, cách thực hiện giống với phòng chống XSS, và các kí tự đặc biệt(DROP, DELETE liên quan đến database).
- Không hiển thị exception, message lỗi: Hacker dựa vào message lỗi để tìm ra cấu trúc database. Khi có lỗi, ta chỉ hiện thông báo lỗi chứ đừng hiển thị đầy đủ thông tin về lỗi, tránh hacker lợi dụng(Đừng quên tắt DEBUG trên production)
- Phân quyền rõ ràng trong database: Nếu chỉ truy cập dữ liệu từ một số bảng, hãy tạo một account trong database, gán quyền truy cập cho account đó chứ đừng dùng account root. Lúc này, dù hacker có inject được sql cũng không thể đọc dữ liệu từ các bảng chính, sửa hay xoá dữ liệu.
- Backup dữ liệu thường xuyên: Cần thận không bao giờ thừa, cứ phòng hơn chống.


# HTTPS
Khi bạn triển khai trang web của mình trên HTTP, tất cả dữ liệu sẽ được trao đổi ở đây sẽ được gửi dưới dạng văn bản thuần túy. Bất cứ ai có kế hoạch ăn cắp có thể thực hiện nó trong quá trình truyền. Vì vậy, để bảo vệ tất cả thông tin có trên ứng dụng web của bạn, bạn nên triển khai nó trên HTTPS.

Trong Laravel bạn có thể dễ dàng làm chuyện này:

```php
namespace App\Providers;
use URL;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        if(config('app.env') === 'production')
            URL::forceScheme('https');
    }
}
```

# Rate Limit Requests
Rate Limit là một phương phát để ngăn chặn vệc tấn công DDoS bằng việc cố gắng gửi nhiều request cùng lúc.

Trong laravel cung cấp một middleware là throttle. Chẳng hạn nếu bạn gửi 1 request lên server qua API thì sẽ nhận được 1 response trong headers là:

```php
Route::group(['prefix' => 'api', 'middleware' => 'throttle'], function () {
    Route::get('people', function () {
        return Person::all();
    });
});
```
// Call api/people
```sh
HTTP/1.1 200 OK
... other headers here ...
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 59
```
Nghĩa là request đã thành công và sẽ còn 59 request nữa trong 1 phút cho chúng ta call. Vậy nếu trong 1 phút ta call vượt 60 thì sao?

```sh
HTTP/1.1 429 Too Many Requests
... other headers here ...
Retry-After: 60
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
```
Không được vui cho lắm nhỉ? Qúa nhiều request. Nếu đợi sau 30 giây nữa mới call tiếp thì sẽ thế nào?

```sh
HTTP/1.1 429 Too Many Requests
... other headers here ...
Retry-After: 30
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
```

Phản hồi tương tự, ngoại trừ bộ đếm thời gian Retry-After cho chúng ta biết thời gian chờ đợi đã giảm xuống 30 giây.

Bạn có thể xem thêm cách sử dụng trong tài liệu của Laravel.

# Replay Attack

Một vụ tấn công phát lại - replay attack, hay còn gọi là ‘playback attack’, là một hình thức tấn công mạng lưới trong đó các thực thể độc hại chặn và lặp lại việc truyền tải một dữ liệu hợp lệ đi vào trong mạng lưới. Nhờ có tính hợp lệ của dữ liệu ban đầu (thường đến từ người dùng đã được cấp quyền), các giao thức bảo mật của mạng lưới sẽ xử lý vụ việc tấn công này chỉ giống như một hình thức truyền tải dữ liệu thông thường.Do các tệp tin ban đầu  đã bị ngăn chặn và được truyền tải lại nguyên văn nên hacker thực hiện vụ tấn công sẽ không cần giải mã chúng.

Các bạn xem thêm định nghĩa và các nguy hiểm của cuộc tấn công này tại đây.
https://academy.binance.com/vi/security/what-is-a-replay-attack

CÁCH PHÒNG CHỐNG:

Trong mỗi request gửi lên bạn thêm 1 tham số là timestamp vào header ứng với thời gian thực hiện mỗi request. Ta tạo một middleware với tham số expire_time là khoảng thời gian mà ứng dụng cho phép mỗi requet có thời hạn trong một khoảng thời gian. Để giảm thiểu cuộc tấn công này thì expire_time phải đủ ngắn (cỡ vài giây chẳng hạn).


Ngoài các lỗi trên thì chắc chắn còn nhiều lỗi khác trong quá trình triển khai nữa, bạn có thể tham khảo thêm ở một số bài viết sau:

https://laravel-news.com/the-laravel-security-checklist-sponsor
https://aglowiditsolutions.com/blog/laravel-security-best-practices/


https://medium.com/@buihuycuong/c%C3%A1c-l%E1%BB%97i-b%E1%BA%A3o-m%E1%BA%ADt-trong-laravel-v%C3%A0-c%C3%A1ch-ph%C3%B2ng-ch%E1%BB%91ng-5eac12a088bc