# BÀI TẬP 3 - MÔN PHÁT TRIỂN ỨNG DỤNG TRÊN THIẾT BỊ DI DỘNG
# NGUYỄN NGUYỆT LINH - K225480106039

---

## I. KIẾN TRÚC ỨNG DỤNG & CÔNG NGHỆ LỰA CHỌN

Thay vì sử dụng các công cụ kéo thả kéo theo nhiều mã nguồn rác như MIT App Inventor, ứng dụng này được xây dựng trên nền tảng **Android Native sử dụng ngôn ngữ Java**. Kiến trúc hệ thống tập trung vào 3 thành phần cốt lõi:

| Thành phần | Công nghệ sử dụng | Tại sao lại dùng công nghệ này? |
| :--- | :--- | :--- |
| **Giao diện (UI)** | `LinearLayout` (Vertical) & XML Resources | Tối ưu hóa hiệu năng render. Việc tách biệt `strings.xml` và `colors.xml` giúp mã nguồn XML sạch, dễ bảo trì và hỗ trợ đa ngôn ngữ sau này. |
| **Xử lý Mạng (Networking)** | Thư viện **Retrofit 2** & **Gson Converter** | Là chuẩn công nghiệp (Best Practice) trên Android. Giúp tự động hóa quá trình chuyển đổi đối tượng Java sang chuỗi JSON và ngược lại, giải phóng lập trình viên khỏi việc viết các hàm cấu trúc chuỗi HTTP thủ công bằng `HttpURLConnection` phức tạp. |
| **Nhúng Web (Web-view)** | `WebView` kết hợp `WebSettings` | Cho phép tích hợp trực tiếp không gian Web của thầy vào luồng trải nghiệm của App mà không cần kích hoạt trình duyệt bên thứ ba (như Chrome), giữ chân người dùng trong ứng dụng. |

---

## II. GIỚI THIỆU ĐỀ TÀI PHÁT TRIỂN APP BMI

### Màn hình 1: Điều hướng & Quản lý Thông tin (`Activity1`)
<img width="959" height="355" alt="image" src="https://github.com/user-attachments/assets/66a6e8d6-e69d-4b47-80ea-f4d4bd3c9687" />

* **Cách thực hiện:** Thiết kế giao diện About bằng XML với các ID nút rõ ràng (`btnGoToActivity2`, `btnGoToActivity3`).
* **Bản chất Code:** Trong `Activity1.java`, sử dụng cơ chế lắng nghe sự kiện `setOnClickListener` kết hợp với **`Intent`**.
* **Tại sao làm vậy?** `Intent` đóng vai trò là một "bưu tá" trong hệ điều hành Android, mang thông điệp yêu cầu hệ thống khởi tạo và cấp phát tài nguyên cho một Activity mới đè lên Activity hiện tại.

---

### Màn hình 2: Bài toán số thực BMI & Gọi API Kết quả (`Activity2`)
Đây là màn hình phức tạp nhất, yêu cầu xử lý logic toán học và tương tác mạng.

<img width="959" height="363" alt="image" src="https://github.com/user-attachments/assets/4e6f114c-e15c-4374-88a1-cfb8b1e339ab" />

#### 1. Sửa đổi để hỗ trợ số thập phân (Yêu cầu thực tế)
* **Trở ngại ban đầu:** Thuộc tính `android:inputType="number"` chỉ cho phép bàn phím ảo nhập số nguyên.
* **Giải pháp thực hiện:** Chuyển cấu hình XML thành `android:inputType="numberDecimal"`. Lúc này, hệ thống sẽ mở rộng bộ mã bàn phím, cho phép người dùng gõ thêm dấu chấm (`.`).
* **Xử lý mã nguồn:** Trong Java, thay vì ép kiểu thô bằng `Integer.parseInt()`, mã nguồn bắt buộc phải chuyển sang **`Double.parseDouble()`** để đọc toàn vẹn giá trị có phần thập phân (ví dụ: `65.5` kg hoặc `1.70` m).

#### 2. Kỹ thuật chuyển đổi cấu trúc JSON lồng nhau gửi lên Server
Yêu cầu API của thầy có cấu trúc JSON lồng nhau dạng đối tượng (Object):
```json
{ 
  "app_by": "MSV", 
  "input": { "a": 1.7, "b": 65.5, "c": 20, "name": "hello tắc kè" }, 
  "output": { "ketluan": "Bình thường", "abc": "BMI_Calculator", "nghiem": 21.38 } 
}
```

Cách thực hiện: Để xử lý cấu trúc này bằng Retrofit 2, tôi tiến hành xây dựng các lớp Java POJO (Plain Old Java Object) mô phỏng chính xác cấu trúc dữ liệu mang tên MathRequest.java.

Tại sao làm vậy? Thư viện GsonConverterFactory được cấu hình đi kèm với Retrofit sẽ tự động quét qua các thuộc tính của đối tượng MathRequest, phân tích kiểu dữ liệu (chuỗi, số thực, số nguyên) để tự động "đóng gói" (Serialize) thành chuỗi JSON chuẩn mà Server yêu cầu mà không cần lập trình viên can thiệp thủ công.

#### 3. Cơ chế gọi API Bất đồng bộ (Asynchronous Call)
Bản chất mã nguồn: Sử dụng phương thức .enqueue(new Callback<MathResponse>()).

Tại sao phải dùng .enqueue thay vì .execute? Trong Android, tất cả các tác vụ liên quan đến mạng (Network) tuyệt đối không được chạy trên Luồng chính (Main Thread/UI Thread) vì sẽ gây treo ứng dụng (lỗi NetworkOnMainThreadException). Hàm .enqueue của Retrofit sẽ tự động đẩy tác vụ gọi API xuống một Luồng phụ (Background Thread) để xử lý ngầm, khi nào có kết quả từ Server trả về thì mới đẩy ngược dữ liệu về Main Thread để hiển thị Toast thông báo cho người dùng.

### Màn hình 3: WebView Tích hợp Log Sinh viên (Activity3)
Cách thực hiện: Khai báo một thẻ <WebView> chiếm toàn bộ kích thước màn hình trong activity_3.xml.

<img width="959" height="355" alt="image" src="https://github.com/user-attachments/assets/78071c78-f9bf-418d-be72-98bf6294c9ab" />

Bản chất Code:

```Java
WebSettings webSettings = webView.getSettings();
webSettings.setJavaScriptEnabled(true);
webView.setWebViewClient(new WebViewClient());
```

Tại sao phải cấu hình như thế này? * Định dạng trang web của thầy (https://k58kmt.tdh.io.vn) có chứa các mã script động để bắt log mã số sinh viên. Nếu không bật setJavaScriptEnabled(true), trang web sẽ bị vô hiệu hóa tính năng xử lý mã lệnh và trở thành trang tĩnh lỗi.

Việc gán một WebViewClient mới giúp ghi đè cơ chế mở link của Android, ép hệ thống phải render trang web ngay inside vùng hiển thị của App thay vì kích hoạt một ứng dụng Browser ngoài.

## III. CÁC LƯU Ý KỸ THUẬT VÀ KINH NGHIỆM XỬ LÝ LỖI (QUYẾT ĐỊNH ĐIỂM SỐ)

Trong quá trình chuyển đổi từ tư duy kéo thả sang lập trình mã nguồn trực tiếp trên Android Studio, có những lỗi hệ thống rất đặc thù cần phải đặc biệt lưu ý:

### 1. Lỗi Biên dịch Tài nguyên AAPT (`ParsedResource`)
* **Hiện tượng:** Ứng dụng báo lỗi đỏ toàn bộ và không thể biên dịch, dòng thông báo lỗi trỏ về các tiến trình của `aaptcompiler`.
* **Nguyên nhân:** Do viết sai quy tắc cấu pháp của file XML hệ thống. Ví dụ:
  * Thiếu dấu `#` trước các mã màu Hex trong file `colors.xml` (Hệ thống yêu cầu bắt buộc dạng `#RRGGBB`).
  * Viết trực tiếp ký tự đặc biệt `&` trong file `strings.xml` mà quên không mã hóa thành quy chuẩn `&amp;`.
* **Kinh nghiệm:** Trình biên dịch tài nguyên của Android (`AAPT`) cực kỳ nghiêm ngặt. Khi gặp lỗi này, cần rà soát ngay các file cấu hình trong thư mục `res/values/`.
<img width="959" height="302" alt="image" src="https://github.com/user-attachments/assets/4662c3a4-7eb6-4b84-b473-b5060adface5" />

### 2. Lỗi Bảo mật Kết nối Mạng (`Cleartext Traffic`)
* **Hiện tượng:** `WebView` tải trang trắng xóa (không hiển thị nội dung) hoặc thư viện `Retrofit` trả về lỗi kết nối không an toàn (Failed to connect).
* **Nguyên nhân:** Kể từ phiên bản Android 9 (API 28) trở lên, Google mặc định chặn toàn bộ các kết nối dạng văn bản thuần túy HTTP không mã hóa nhằm bảo mật thông tin người dùng.
* **Giải pháp khắc phục:** Bắt buộc phải khai báo bổ sung thuộc tính `android:usesCleartextTraffic="true"` bên trong thẻ `<application>` của file cấu hình hệ thống `AndroidManifest.xml`.

### 3. Kỹ thuật Đồng bộ Gradle và Bộ nhớ đệm (`Sync & Clean`)
* **Hiện tượng:** Code Java báo lỗi chữ đỏ hàng loạt ở các vùng tham chiếu như `R.layout...` hoặc `Cannot resolve symbol` dù kiểm tra tên file hoàn toàn đúng.
* **Nguyên nhân:** Android Studio quản lý thư viện và ánh xạ tài nguyên qua một file trung gian (`R.java`) tự động sinh ra. Khi chỉnh sửa file XML hoặc cấu hình cài đặt thư viện mạng, bộ nhớ đệm (Cache) của IDE chưa kịp quét và cập nhật lại mã nguồn.
* **Quy trình xử lý dứt điểm:**
  1. Trên thanh công cụ, chọn **Build** $\rightarrow$ **Clean Project** (Xóa toàn bộ file build lỗi cũ).
  2. Tiếp tục chọn **Build** $\rightarrow$ **Rebuild Project** (Ép hệ thống quét và tái cấu trúc lại toàn bộ tài nguyên).

## IV. QUY TRÌNH THỰC HIỆN CHI TIẾT

### Bước 1: Thiết lập cấu hình Hệ thống và Thư viện

Để ứng dụng có thể kết nối mạng và sử dụng các thư viện hỗ trợ, quy trình thiết lập ban đầu được thực hiện như sau:

#### 1. Cấp quyền Internet trong Hệ thống
Khai báo thẻ xin quyền truy cập Internet trong file `AndroidManifest.xml`. Thẻ này phải đặt nằm phía trên thẻ `<application>` để hệ điều hành Android cấp quyền chạy môi trường mạng cho ứng dụng:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
#### 2. Kích hoạt kết nối HTTP/HTTPS tùy biến
Kể từ phiên bản Android cao, hệ thống mặc định chặn các liên kết không mã hóa bảo mật. Cần thêm trực tiếp thuộc tính android:usesCleartextTraffic="true" vào bên trong thẻ <application> để tránh lỗi chặn mã hóa, giúp WebView tải trang của thầy mượt mà:

```XML
<application
    android:usesCleartextTraffic="true"
    ... >
```

#### 3. Khai báo thư viện mạng trong hệ thống Build
Mở file cấu hình build.gradle (Module :app) hoặc build.gradle.kts (Module :app) và thêm hai gói thư viện hỗ trợ giao tiếp API kết hợp chuyển đổi dữ liệu JSON tự động vào khối dependencies:

```Groovy
dependencies {
    // Thư viện xử lý mạng Retrofit 2
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    // Bộ chuyển đổi tự động Object Java sang JSON (Gson)
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
}
```
<img width="959" height="350" alt="image" src="https://github.com/user-attachments/assets/5b375127-2dce-4285-b350-ec08298214d0" />

#### 4. Khai báo và định tuyến các màn hình (Activities)
Đảm bảo cả 3 màn hình Activity1, Activity2, Activity3 đều được định nghĩa đầy đủ cấu trúc thẻ bên trong AndroidManifest.xml. Trong đó, Activity1 được gắn bộ lọc LAUNCHER để hệ thống nhận diện làm màn hình chính chạy đầu tiên khi mở app:

```XML
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BMI"
        android:usesCleartextTraffic="true"
        tools:targetApi="34">

        <activity
            android:name=".Activity1"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity
            android:name=".Activity2"
            android:exported="false" />

        <activity
            android:name=".Activity3"
            android:exported="false" />

    </application>

</manifest>
```
### Bước 2: Quản lý Tài nguyên Tập trung (`strings.xml` & `colors.xml`)

Việc quản lý tài nguyên tập trung giúp tách biệt hoàn toàn phần chữ/màu sắc ra khỏi file giao diện XML, tạo tiền đề để bảo trì code và phát triển đa ngôn ngữ.

* **Chuẩn hóa văn bản (`strings.xml`):** Toàn bộ chuỗi ký tự hiển thị (tên ứng dụng, nhãn trên nút bấm, gợi ý nhập liệu) đều được quy hoạch tập trung. Đối với các ký tự đặc biệt có tính logic trong cú pháp XML như ký tự `&`, bắt buộc phải được mã hóa an toàn thành dạng thực thể định dạng: `&amp;`.
* **Quản lý hệ thống màu (`colors.xml`):** Toàn bộ mã màu sử dụng cho thành phần đồ họa hoặc văn bản trong ứng dụng được cấu trúc đồng bộ. Các mã màu phải tuân thủ nghiêm ngặt định dạng mã màu Hex và có dấu thăng `#` ở đầu (Ví dụ chủ đạo: `#3700B3`).

<img width="959" height="257" alt="image" src="https://github.com/user-attachments/assets/5d87b581-3c50-4933-8915-0b28be685564" />

<img width="959" height="342" alt="image" src="https://github.com/user-attachments/assets/88f605f1-fd20-405a-9b61-f5c4ac419d20" />

---

### Bước 3: Thiết kế Giao diện XML (Layouts)

Xây dựng các tệp tin Layout trong thư mục `res/layout/` để định hình khung nhìn của 3 màn hình.

* **Cấu trúc phân tầng trực quan:** Sử dụng đối tượng `LinearLayout` làm gốc chủ đạo cho các màn hình, kết hợp thuộc tính định hướng `android:orientation="vertical"`. Cấu hình này giúp hệ thống tự động sắp xếp các thành phần giao diện (TextView, EditText, Button) nối tiếp nhau theo chiều dọc từ trên xuống dưới một cách tuần tự mà không bị xô lệch trên các kích thước màn hình khác nhau.
* **Tối ưu hóa bộ gõ thập phân:** Riêng các ô nhập liệu `EditText` dành cho Chiều cao và Cân nặng, thuộc tính `android:inputType` được cấu hình chính xác là `"numberDecimal"`. Thiết lập này giúp hệ thống mở bàn phím ảo dạng số tích hợp phím dấu chấm `.`, hỗ trợ người dùng nhập số thực dễ dàng thay vì chỉ nhập được số nguyên như mặc định.
<img width="959" height="539" alt="image" src="https://github.com/user-attachments/assets/05d1b415-8d26-46b7-8ebf-eae39007daa6" />

---

### Bước 4: Xây dựng Cấu trúc Dữ liệu (Models) và Logic Gọi API

Chuyển đổi luồng xử lý từ dữ liệu cục bộ sang môi trường mạng để đồng bộ hóa kết quả lên hệ thống Server của giảng viên.

#### 1. Định nghĩa cấu trúc dữ liệu POJO (Plain Old Java Object)
Tạo ra hai file cấu trúc lớp đối tượng là `MathRequest.java` và `MathResponse.java` nhằm ánh xạ 1:1 với định dạng chuỗi JSON của Server. Để đáp ứng việc gửi bài toán dạng số thực (nhập chiều cao, cân nặng thập phân), hai thuộc tính `a` và `b` nằm bên trong lớp thực thể con `InputData` bắt buộc phải cấu hình kiểu dữ liệu **`double`**.
<img width="959" height="484" alt="image" src="https://github.com/user-attachments/assets/5e5d94ca-a2c5-43e6-a007-c1773ec6203c" />

```java
package com.example.bmi; // Thay bằng package thực tế của bạn nếu khác

public class MathRequest {
    private String app_by;
    private InputData input;
    private OutputData output;

    public MathRequest(String app_by, InputData input, OutputData output) {
        this.app_by = app_by;
        this.input = input;
        this.output = output;
    }

    public static class InputData {
        double a, b; // Đổi sang kiểu double để nhận số thập phân
        int c;       // Tuổi giữ nguyên int
        String name;

        public InputData(double a, double b, int c, String name) {
            this.a = a; this.b = b; this.c = c; this.name = name;
        }
    }

    public static class OutputData {
        String ketluan, abc;
        double nghiem;
        public OutputData(String ketluan, String abc, double nghiem) {
            this.ketluan = ketluan; this.abc = abc; this.nghiem = nghiem;
        }
    }
}
```
#### 2. Quản lý Endpoint thông qua Interface
Tạo một Interface mang tên `ApiService.java` đóng vai trò làm bộ khung ánh xạ các phương thức HTTP. Tại đây, sử dụng Annotation `@POST("api")` để ký hiệu cho thư viện mạng biết ứng dụng sẽ thực hiện phương thức gửi gói tin lên endpoint `/api`.
<img width="959" height="284" alt="image" src="https://github.com/user-attachments/assets/9d0dcc16-f934-46fe-bb63-982caf56e3eb" />

```java
package com.example.bmi;

import retrofit2.Call;
import retrofit2.http.Body;
import retrofit2.http.POST;

public interface ApiService {
    @POST("api")
    Call<MathResponse> sendMathResult(@Body MathRequest request);
}
```
#### 3. Xử lý logic tính toán và kích hoạt luồng truyền tải
Trong file điều khiển logic chính `Activity2.java`:
* Khai báo hàm `Double.parseDouble()` để chuyển đổi văn bản thô từ ô nhập liệu về định dạng số thực.
* Thực hiện thuật toán tính toán chỉ số cơ thể BMI theo công thức chuẩn và gán kết luận phân loại.
* Khởi tạo đối tượng truyền tải mạng `Retrofit` kết hợp bộ chuyển đổi tự động `GsonConverterFactory`.
* Sử dụng phương thức gọi bất đồng bộ **`.enqueue()`** thay vì gọi đồng bộ để đẩy tiến trình gửi gói dữ liệu JSON xuống luồng chạy ngầm (Background Thread), giúp luồng xử lý giao diện chính (Main Thread) không bị đóng băng hay gây treo ứng dụng.
<img width="959" height="473" alt="image" src="https://github.com/user-attachments/assets/a19df0af-e9c5-40f0-a9c3-76881ee7c1d0" />

```java
package com.example.bmi;

import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;
import android.widget.Toast;
import androidx.appcompat.app.AppCompatActivity;
import java.util.Locale;
import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

public class Activity2 extends AppCompatActivity {
    EditText edtChieuCao, edtCanNang, edtTuoi;
    Button btnTinhBMI;
    TextView txtKetQuaBMI;
    String maSv = "K225480106039"; // <--- Điền mã SV của bạn vào đây

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_2);

        // Ánh xạ bằng findViewById cũ của ông, giữ nguyên không đổi
        edtChieuCao = findViewById(R.id.edtChieuCao);
        edtCanNang = findViewById(R.id.edtCanNang);
        edtTuoi = findViewById(R.id.edtTuoi);
        btnTinhBMI = findViewById(R.id.btnTinhBMI);
        txtKetQuaBMI = findViewById(R.id.txtKetQuaBMI);

        btnTinhBMI.setOnClickListener(v -> tinhToanVaGuiAPI());
    }

    private void tinhToanVaGuiAPI() {
        String strChieuCao = edtChieuCao.getText().toString();
        String strCanNang = edtCanNang.getText().toString();
        String strTuoi = edtTuoi.getText().toString();

        if (strChieuCao.isEmpty() || strCanNang.isEmpty() || strTuoi.isEmpty()) {
            Toast.makeText(this, "Vui lòng điền đủ thông tin a, b, c!", Toast.LENGTH_SHORT).show();
            return;
        }

        // CHỖ THAY ĐỔI: Chuyển sang Double.parseDouble để nhận số thập phân
        double a = Double.parseDouble(strChieuCao); // Chiều cao (ví dụ: 1.70 hoặc 170)
        double b = Double.parseDouble(strCanNang);  // Cân nặng (ví dụ: 65.5)
        int c = Integer.parseInt(strTuoi);          // Tuổi giữ nguyên số nguyên

        // Tự động xử lý: Nếu nhập cm (170) thì chia 100, nếu nhập mét (1.7) thì giữ nguyên
        double chieuCaoMet = (a > 10) ? (a / 100.0) : a;

        // Công thức tính BMI
        double bmi = b / (chieuCaoMet * chieuCaoMet);
        bmi = Math.round(bmi * 100.0) / 100.0; // Làm tròn lấy 2 chữ số sau dấu phẩy

        String ketLuan;
        if (bmi < 18.5) {
            ketLuan = "Gầy (Thiếu cân)";
        } else if (bmi >= 18.5 && bmi < 24.9) {
            ketLuan = "Bình thường";
        } else if (bmi >= 25 && bmi < 29.9) {
            ketLuan = "Thừa cân";
        } else {
            ketLuan = "Béo phì";
        }

        String textHienThi = String.format(Locale.getDefault(), "Chỉ số BMI: %.2f\nĐánh giá: %s", bmi, ketLuan);
        txtKetQuaBMI.setText(textHienThi);

        // --- GỌI API RETROFIT ---
        Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("https://k58kmt.tdh.io.vn/")
                .addConverterFactory(GsonConverterFactory.create())
                .build();

        ApiService apiService = retrofit.create(ApiService.class);

        // Đóng gói data gửi đi (nhớ sửa file MathRequest.java biến a, b thành double như hướng dẫn trước là được)
        MathRequest.InputData input = new MathRequest.InputData(a, b, c, "hello tắc kè");
        MathRequest.OutputData output = new MathRequest.OutputData(ketLuan, "BMI_Calculator", bmi);
        MathRequest requestData = new MathRequest(maSv, input, output);

        apiService.sendMathResult(requestData).enqueue(new Callback<MathResponse>() {
            @Override
            public void onResponse(Call<MathResponse> call, Response<MathResponse> response) {
                if (response.isSuccessful() && response.body() != null) {
                    MathResponse res = response.body();
                    Toast.makeText(Activity2.this, "Gửi API thành công! STT: " + res.getStt(), Toast.LENGTH_LONG).show();
                } else {
                    Toast.makeText(Activity2.this, "Lỗi xử lý phản hồi dữ liệu", Toast.LENGTH_SHORT).show();
                }
            }

            @Override
            public void onFailure(Call<MathResponse> call, Throwable t) {
                Toast.makeText(Activity2.this, "Lỗi mạng: " + t.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });
    }
}
```
---

## III. CÁC CHÚ Ý QUAN TRỌNG KHI TRIỂN KHAI (KINH NGHIỆM XỬ LÝ LỖI)

Trong quá trình dịch chuyển tư duy từ công cụ kéo thả kéo theo nhiều mã nguồn rác như MIT App Inventor sang lập trình mã nguồn trực tiếp trên phần mềm Android Studio, có những lỗi hệ thống rất đặc thù mà lập trình viên cần kiểm soát:

### 📌 1. Lỗi Cú pháp nghiêm ngặt trong File Tài nguyên (`colors.xml`)
* **Bản chất hệ thống:** Mã màu Hex trong môi trường Android bắt buộc phải có dấu thăng `#` ở ký tự đầu tiên (ví dụ: `#000000`).
* **Hệ quả:** Nếu vô tình thiếu dấu `#` hoặc gõ các ký tự nằm ngoài hệ Hex (0-9 và A-F) như chữ `I`, chữ `G`,... hệ thống biên dịch tài nguyên `AAPT` của Android Studio sẽ lập tức đổ gãy, báo lỗi `Can not extract resource...` và khóa chặt toàn bộ tiến trình Build ứng dụng.

### 📌 2. Đồng bộ kiểu dữ liệu số thực khi Ép kiểu (Thập phân)
* **Bản chất hệ thống:** Mặc định ban đầu bài toán mẫu chỉ nhận các tham số dạng số nguyên (`int`). 
* **Hệ quả:** Khi chuyển đổi ứng dụng thực tế để nhận số thập phân (như cân nặng `65.5` kg), lập trình viên bắt buộc phải thực hiện đồng bộ đổi kiểu dữ liệu sang `double` ở cả hai nơi độc lập: File tính toán logic (`Activity2.java`) và File định dạng lớp truyền tải gói tin (`MathRequest.java`). Nếu xảy ra hiện tượng lệch kiểu dữ liệu giữa hai file, thư viện chuyển đổi GSON sẽ đóng gói sai cấu trúc dữ liệu hoặc ứng dụng sẽ lập tức bị crash khi người dùng bấm nút tính toán.

### 📌 3. Cơ chế Lưu Bộ nhớ đệm Hệ thống (`Clean & Rebuild Project`)
* **Bản chất hệ thống:** Mỗi khi lập trình viên thực hiện thay đổi mã cấu hình hệ thống (như thêm thư viện mạng trong file `build.gradle`) hoặc sửa các lỗi cú pháp trong thư mục tài nguyên `values`, Android Studio thường có xu hướng lưu lại bộ nhớ đệm (Cache) của phiên làm việc lỗi trước đó.
* **Giải pháp dứt điểm:** Hành động bắt buộc cần làm là truy cập lên thanh công cụ phía trên của phần mềm, chọn mục **Build** $\rightarrow$ kích hoạt lệnh **Clean Project** (Xóa sạch thư mục build lỗi cũ), sau đó tiếp tục chọn **Build** $\rightarrow$ kích hoạt lệnh **Rebuild Project** để ép toàn bộ hệ thống quét và biên dịch lại mã nguồn sạch từ đầu.

### 📌 4. Cơ chế Kiểm soát và Nhúng Web (`WebView`)
* **Kích hoạt Script:** Để trang web kết quả hiển thị của giảng viên (`https://k58kmt.tdh.io.vn`) chạy đầy đủ các tính năng tương tác tự động bắt log mã số sinh viên, bắt buộc phải kích hoạt môi trường JavaScript bằng câu lệnh cấu hình: `webSettings.setJavaScriptEnabled(true);`.
* **Điều hướng luồng nội bộ:** Đồng thời phải thiết lập hàm `webView.setWebViewClient(new WebViewClient());`. Việc gán một Client mới này có tác dụng ghi đè lên trình duyệt mặc định của hệ điều hành, ép trang web luôn luôn phải render nội dung ngay bên trong không gian hiển thị của ứng dụng, tránh hiện tượng khi người dùng tương tác liên kết, trang web bị văng ra trình duyệt Chrome bên ngoài điện thoại.

# Một số hình ảnh về app

<img width="300" height="720" alt="Screenshot_2026-06-09-12-50-43-760_com example bmi" src="https://github.com/user-attachments/assets/9cc20179-02b1-4c24-9125-3831d5d7589c" />

<img  width="300" height="720" alt="Screenshot_2026-06-09-12-51-15-102_com example bmi" src="https://github.com/user-attachments/assets/1731a1d5-e8c5-40d4-90df-6f108b8087d5" />

<img  width="300" height="720" alt="Screenshot_2026-06-09-12-51-25-260_com example bmi" src="https://github.com/user-attachments/assets/fc9bb940-67f8-4110-a0b4-b4e533c2b164" />
