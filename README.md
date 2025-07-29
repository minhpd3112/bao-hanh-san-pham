# **Dự Án Đăng Ký Bảo Hành Phụ Tùng Ô Tô Tương Tác**

Dự án này cung cấp một giải pháp đăng ký bảo hành điện tử hiện đại và thân thiện với người dùng cho các sản phẩm phụ tùng ô tô. Khách hàng có thể quét mã QR từ tem bảo hành để truy cập một biểu mẫu web đẹp mắt, điền thông tin và kích hoạt bảo hành, với dữ liệu tự động được lưu trữ vào Google Sheet.

## **Mục Lục**

1. [Tổng Quan Dự Án]  
2. [Các Thành Phần Chính]  
3. [Hướng Dẫn Triển Khai Chi Tiết]
   * [Bước 1: Chuẩn Bị Google Sheet]
   * [Bước 2: Tạo Google Apps Script]
   * [Bước 3: Cập Nhật Mã HTML của Biểu Mẫu]
   * [Bước 4: Triển Khai Biểu Mẫu Lên GitHub Pages]
4. [Cách Thức Hoạt Động]
5. [Tùy Chỉnh & Mở Rộng]

## **Tổng Quan Dự Án**

Mục tiêu của dự án là thay thế các biểu mẫu bảo hành truyền thống bằng một trải nghiệm số hóa, tiện lợi và chuyên nghiệp. Khi khách hàng quét mã QR trên sản phẩm, họ sẽ được dẫn đến một trang web (SPA \- Single-Page Application) để điền thông tin cá nhân, thông tin gara và mã sản phẩm. Dữ liệu này sau đó sẽ được tự động ghi vào một Google Sheet để quản lý.

## **Các Thành Phần Chính**

Dự án bao gồm ba thành phần chính hoạt động cùng nhau:

1. **Google Sheet:** Đóng vai trò là cơ sở dữ liệu để lưu trữ tất cả các thông tin đăng ký bảo hành.  
2. **Google Apps Script:** Một đoạn mã JavaScript chạy trên nền tảng Google, hoạt động như một API trung gian. Nó nhận dữ liệu từ biểu mẫu web và ghi vào Google Sheet.  
3. **Biểu Mẫu HTML Tương Tác (SPA):** Giao diện người dùng mà khách hàng tương tác. Được xây dựng với HTML, Tailwind CSS (để thiết kế responsive) và JavaScript (để xử lý logic, dropdown động cho Tỉnh/Thành phố & Quận/Huyện, và gửi dữ liệu).

## **Hướng Dẫn Triển Khai Chi Tiết**

Hãy làm theo các bước dưới đây để thiết lập và triển khai hệ thống của bạn.

### **Bước 1: Chuẩn Bị Google Sheet**

1. **Tạo Google Sheet mới:**  
   * Truy cập [sheets.google.com](https://sheets.google.com/).  
   * Tạo một bảng tính trống mới và đặt tên, ví dụ: "Dữ liệu Kích Hoạt Bảo Hành Phụ Tùng".  
2. **Thiết lập tiêu đề cột:**  
   * Tại hàng đầu tiên (hàng tiêu đề), nhập chính xác các tên cột sau vào các ô từ A1 trở đi:  
     * A1: Timestamp  
     * B1: Họ và Tên  
     * C1: Số điện thoại  
     * D1: Tỉnh/Thành phố  
     * E1: Quận/Huyện  
     * F1: Tên Gara/Đại lý  
     * G1: Mã Sản Phẩm

### **Bước 2: Tạo Google Apps Script**

Đây là phần quan trọng để kết nối biểu mẫu HTML với Google Sheet.

1. **Mở Apps Script:**  
   * Trong Google Sheet bạn vừa tạo, vào menu **Tiện ích mở rộng (Extensions)** \> **Apps Script**.  
   * Một tab mới sẽ mở ra với trình soạn thảo mã.  
2. **Dán mã Apps Script:**  
   * Xóa bất kỳ mã nào có sẵn và dán toàn bộ đoạn mã dưới đây vào trình soạn thảo:

function doPost(e) {  
  const sheetName \= 'Sheet1'; // Đảm bảo tên sheet này khớp với tên sheet trong Google Sheet của bạn (thường là Sheet1 mặc định)  
  const scriptProp \= PropertiesService.getScriptProperties();  
  const url \= scriptProp.getProperty('key');  
  const lock \= LockService.getScriptLock();  
  lock.waitLock(10000); 

  try {  
    // Lấy ID của Google Sheet từ URL của Sheet (ví dụ: https://docs.google.com/spreadsheets/d/YOUR\_SPREADSHEET\_ID\_HERE/edit)  
    const doc \= SpreadsheetApp.openById('YOUR\_SPREADSHEET\_ID\_HERE');   
    const sheet \= doc.getSheetByName(sheetName);

    const headers \= sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()\[0\];  
    const nextRow \= sheet.getLastRow() \+ 1;

    const rowData \= \[\];  
    rowData.push(new Date()); // Timestamp (ngày/giờ gửi form)

    // Lấy dữ liệu từ form HTML (đảm bảo tên 'name' trong input HTML khớp với các tham số này)  
    rowData.push(e.parameter.fullName);  
    rowData.push(e.parameter.phone);  
    rowData.push(e.parameter.province);  
    rowData.push(e.parameter.district);  
    rowData.push(e.parameter.garage);  
    rowData.push(e.parameter.productCode); // Thêm trường Mã Sản Phẩm

    // Ghi dữ liệu vào hàng tiếp theo trong sheet  
    sheet.getRange(nextRow, 1, 1, rowData.length).setValues(\[rowData\]);

    // Trả về phản hồi thành công  
    return ContentService.createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow })).setMimeType(ContentService.MimeType.JSON);  
  } catch (error) {  
    // Trả về phản hồi lỗi nếu có vấn đề  
    return ContentService.createTextOutput(JSON.stringify({ 'result': 'error', 'message': error.message })).setMimeType(ContentService.MimeType.JSON);  
  } finally {  
    lock.releaseLock();  
  }  
}

3. **Thay thế YOUR\_SPREADSHEET\_ID\_HERE:**  
   * Quay lại Google Sheet của bạn.  
   * Sao chép ID của Sheet từ URL trên trình duyệt (phần nằm giữa /d/ và /edit): https://docs.google.com/spreadsheets/d/YOUR\_SPREADSHEET\_ID\_HERE/edit\#gid=0  
   * Dán ID này vào mã Apps Script, thay thế YOUR\_SPREADSHEET\_ID\_HERE.  
4. **Lưu và Triển khai (Deploy) Apps Script:**  
   * Nhấp vào biểu tượng **Lưu dự án (Save project)** (hình đĩa mềm).  
   * Nhấp vào **Triển khai (Deploy)** \> **Triển khai mới (New deployment)**.  
   * Trong cửa sổ "New deployment":  
     * Nhấp vào biểu tượng bánh răng (Select type) và chọn **Web app**.  
     * **Mô tả (Description):** "API Ghi dữ liệu bảo hành".  
     * **Thực thi với tư cách (Execute as):** "Người dùng triển khai (Me)".  
     * **Ai có quyền truy cập (Who has access):** **"Bất kỳ ai (Anyone)"**. (Rất quan trọng để form HTML có thể gửi dữ liệu).  
   * Nhấp **Triển khai (Deploy)**.  
   * Lần đầu tiên, bạn sẽ được yêu cầu cấp quyền. Hãy cấp quyền truy cập cần thiết.  
   * Sau khi triển khai thành công, bạn sẽ nhận được một **URL ứng dụng web (Web app URL)**. **Sao chép URL này lại**, bạn sẽ cần nó ở bước tiếp theo.

### **Bước 3: Cập Nhật Mã HTML của Biểu Mẫu**

Mã HTML này là giao diện người dùng chính.

1. **Lấy mã HTML:** Sử dụng mã HTML.  
2. **Lưu tệp HTML:** Lưu toàn bộ mã đó vào một tệp trên máy tính của bạn với tên ví dụ: baohanh.html.  
3. **Thay thế YOUR\_WEB\_APP\_URL\_HERE:**  
   * Mở tệp baohanh.html bằng trình soạn thảo văn bản.  
   * Tìm dòng const scriptURL \= 'YOUR\_WEB\_APP\_URL\_HERE';  
   * Thay thế YOUR\_WEB\_APP\_URL\_HERE bằng **URL ứng dụng web** mà bạn đã sao chép từ Bước 2\.  
4. **Lưu lại tệp HTML.**

### **Bước 4: Triển Khai Biểu Mẫu Lên GitHub Pages**

Đây là cách để biểu mẫu HTML của bạn có thể truy cập được công khai qua một liên kết.

1. **Tạo kho lưu trữ (Repository) mới:**    
   *Tải tệp HTML lên:* 
2. **Kích hoạt GitHub Pages:**  
   * Trong kho lưu trữ của bạn, nhấp vào tab **"Settings"**.  
   * Trong menu bên trái, nhấp vào **"Pages"**.  
   * Trong phần "Build and deployment" \> "Source", chọn **"Deploy from a branch"**.  
   * Trong phần "Branch", chọn nhánh mà bạn đã tải tệp lên (thường là main hoặc master).  
   * Chọn thư mục gốc là / (root).  
   * Nhấp **"Save"**.  
3. **Lấy liên kết công khai:**  
   * Sau khi lưu, GitHub Pages sẽ bắt đầu triển khai (có thể mất vài phút).  
   * Làm mới trang "Pages" sau một thời gian, bạn sẽ thấy thông báo "Your site is published at..." cùng với **URL công khai** của biểu mẫu.  
   * Đường dẫn sẽ có dạng: https://yourusername.github.io/your-repository-name/baohanh.html

## **Cách Thức Hoạt Động**

1. **Quét mã QR:** Khách hàng quét mã QR trên sản phẩm, mã này sẽ dẫn họ đến URL công khai của biểu mẫu HTML trên GitHub Pages.  
2. **Điền thông tin:** Khách hàng điền đầy đủ các thông tin yêu cầu trên giao diện thân thiện.  
3. **Gửi biểu mẫu:** Khi khách hàng nhấp "Kích Hoạt Bảo Hành", JavaScript trong biểu mẫu sẽ gửi dữ liệu đến URL của Google Apps Script.  
4. **Lưu dữ liệu:** Google Apps Script nhận dữ liệu và tự động ghi vào hàng mới trong Google Sheet của bạn, bao gồm cả dấu thời gian kích hoạt.  
5. **Xác nhận:** Biểu mẫu HTML sẽ hiển thị thông báo "Kích hoạt thành công\!" ngay lập tức cho khách hàng.

## **Tùy Chỉnh & Mở Rộng**

* **Dữ liệu Tỉnh/Thành phố & Quận/Huyện:** Bạn có thể chỉnh sửa đối tượng provincesAndDistricts trong thẻ \<script\> của tệp HTML để thêm hoặc cập nhật danh sách các Quận/Huyện cho các Tỉnh/Thành phố khác.  
* **Thiết kế giao diện:** Bạn có thể tùy chỉnh màu sắc, phông chữ, bố cục và thêm logo của riêng mình bằng cách chỉnh sửa các lớp Tailwind CSS và CSS trong tệp HTML.  
* **Xác thực dữ liệu:** Thêm các quy tắc xác thực phức tạp hơn (ví dụ: định dạng email, độ dài số điện thoại) trong JavaScript của biểu mẫu.  
* **Thông báo nâng cao:** Thay vì alert(), bạn có thể triển khai một modal (hộp thoại) tùy chỉnh để hiển thị thông báo lỗi hoặc thành công.

Chúc bạn thành công với hệ thống đăng ký bảo hành mới này\!
