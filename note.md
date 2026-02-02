**BÁO CÁO PHÂN TÍCH MẪU ĐỘC HẠI**

---

## 1. CFBAFBFIEH.exe

Ngay khi mở file, chương trình hiển thị giao diện đồ họa tự giới thiệu là công cụ kích hoạt Windows/Office, bố cục tương tự các bản activator phổ biến trên diễn đàn crack.

![Giao diện kích hoạt 1](https://raw.githubusercontent.com/AFatc4t/aa/main/image.png)

Giao diện thứ hai đặt các nút kích hoạt song song cho Windows và Office, kèm các nhãn hướng dẫn đơn giản “KMS” nhằm thuyết phục người dùng rằng đây là công cụ đáng tin cậy.

![Giao diện kích hoạt 2](https://raw.githubusercontent.com/AFatc4t/aa/main/image-1.png)

Khi người dùng nhấn nút, binary gọi `cmd.exe` để thực thi script nội bộ; ảnh ghi lại cho thấy các lệnh KMS quen thuộc được bung ra ngay bên trong console, xác nhận đây không phải công cụ mới mà là bản bọc lại.

![Lệnh kích hoạt](https://raw.githubusercontent.com/AFatc4t/aa/main/image-7.png)

Đào sâu hơn, một blob Base64 khổng lồ được giải mã thành Activate.cmd. Ảnh dưới ghi lại đoạn đầu của blob với header đặc trưng của script KMS_VL_ALL.

![Blob Base64 (phần 1)](https://raw.githubusercontent.com/AFatc4t/aa/main/image-6.png)

Phần tiếp theo của blob chứa các biến hằng như `KMS_HWID=0x3A1C049600B60076`, trùng khớp hoàn toàn với mã nguồn công khai của Microsoft Activation Scripts.

![Blob Base64 (phần 2)](https://raw.githubusercontent.com/AFatc4t/aa/main/image-5.png)

Từ các chứng cứ trên có thể kết luận CFBAFBFIEH.exe chỉ là bản repack GUI (nhiều khả năng dùng Lazarus/Delphi) để nhúng trực tiếp logic KMS_VL_ALL. String dump phát hiện thêm các chuỗi KMS_Activat0r_2024, “Office and Windows Activator!” và “Activation Windows/Office”, củng cố giả thuyết đây là HackTool/Activator, dù không có API mạng đáng ngờ. Việc công cụ chỉnh sửa licensing, tạo Defender exclusion và động chạm System32 vẫn khiến endpoint rủi ro cao, đặc biệt vì những bản repack kiểu này dễ bị cài kèm payload phụ. Nên gỡ bỏ hoàn toàn CFBAFBFIEH.exe, kiểm tra Defender exclusion, Task Scheduler, registry liên quan KMS và tuyên truyền cho người dùng tránh tái sử dụng công cụ crack.

Đối với hoạt động điều tra, nên tìm kiếm thêm các chỉ báo như log Event Viewer 12288/12289 của Software Protection Platform, các scheduled task có tên “KMS” hoặc “Activate”. Nếu phát hiện, đội IR nên snapshot lại hệ thống, so sánh hash của Activate.cmd giải mã được với bản chuẩn của MAS để xác định mức độ biến đổi. Tại tầng mạng, dù chưa thấy hạ tầng điều khiển, vẫn nên kiểm soát bất kỳ kết nối outbound nào trùng khớp hostname hard-coded của MAS để tránh bị lợi dụng làm kênh tải script phụ. Về lâu dài, cần cập nhật chính sách bảo mật nội bộ, quy định rõ việc sử dụng phần mềm không bản quyền sẽ bị xem là hành vi vi phạm an toàn thông tin, đồng thời kết hợp giải pháp Application Control để chặn các binary chưa ký như CFBAFBFIEH.exe.

---

## 2. java.exe

Mẫu thứ hai được đặt tên `java.exe` để đánh lừa người dùng rằng đây là thành phần của Java Runtime, nhưng khi kiểm tra cấu trúc PE thấy ngay dấu hiệu bị nén bởi UPX.

![UPX packing](https://raw.githubusercontent.com/AFatc4t/aa/main/image-2.png)

Giải nén xong, chương trình lộ rõ các tham số dòng lệnh như `--donate-level`, `--cpu-priority`... vốn là tùy chọn cấu hình quen thuộc của miner XMRig, chứng minh bản chất khai thác tiền mã hóa.

![Tham số runtime](https://raw.githubusercontent.com/AFatc4t/aa/main/image-3.png)

Đối chiếu chuỗi và hash với nguồn mở trên Internet cho kết quả trùng khớp website chính thức của XMRig, xác nhận đây là phiên bản bị repack và đổi tên để duy trì bám trụ.

![Truy vết XMRig](https://raw.githubusercontent.com/AFatc4t/aa/main/image-4.png)

Như vậy, java.exe được phân loại miner XMRig trá hình: không phá hoại dữ liệu nhưng tiêu tốn CPU/GPU, gây nóng máy và kết nối về pool đào ngoài kiểm soát. Việc bị pack/đổi tên chứng tỏ tác giả cố tình né phát hiện, nên cần rà soát service, scheduled task, registry Run để tìm persistence. Khuyến nghị xóa file và các cơ chế bám, quét hệ thống với AV/EDR, giám sát lưu lượng tới pool đào và thiết lập cảnh báo sử dụng CPU bất thường để phát hiện sớm biến thể mới.

Ngoài ra, nên thu thập thêm dữ liệu từ Performance Monitor để xác định thời điểm spike tài nguyên, kết hợp với log tường lửa nhằm nhận diện địa chỉ pool. Nếu endpoint đã tham gia đào coin trong thời gian dài, cần đánh giá tác động tới tuổi thọ phần cứng và chi phí điện. Đội ngũ vận hành có thể thiết lập kịch bản hunt dựa trên đặc trưng gọi API `CreateToolhelp32Snapshot` hoặc `GetSystemInfo` liên tục của XMRig, đồng thời tạo chữ ký YARA dựa theo chuỗi “donate-level” nhằm chủ động quét toàn domain. Để phòng tái nhiễm, nên cập nhật chính sách least privilege, không cho phép người dùng tự ý cài đặt phần mềm giả dạng Java và áp dụng cập nhật Java chính thức từ Oracle để tránh nhầm lẫn.

---

## 3. FIDAFIEBFC.exe

Mẫu cuối cùng thể hiện nhiều lớp obfuscation ngay từ khi nạp vào IDA/PEiD: các khối mã bị xáo trộn và chèn nhiều nhánh rối khiến việc đọc logic gần như bất khả thi nếu chỉ phân tích tĩnh.

![Obfuscation tổng quan](https://raw.githubusercontent.com/AFatc4t/aa/main/image-9.png)

Đi sâu vào từng hàm, có thể thấy các lệnh đệm, phép toán vô nghĩa và thủ thuật chuyển control flow liên tục, thể hiện rõ tác giả áp dụng kỹ thuật phòng ngừa reverse mạnh tay.

![Chi tiết mã obfuscate](https://raw.githubusercontent.com/AFatc4t/aa/main/image-8.png)

Song song, ảnh trích xuất import cho thấy bộ API `VirtualAlloc`, `OpenFileMappingA`, `DeleteFileA` cùng cơ chế gọi hàm thông qua con trỏ (call EBX). Đây là chuỗi điển hình của loader/stager: cấp phát vùng nhớ thực thi, chia sẻ dữ liệu với tiến trình khác qua Named Pipe hoặc shared memory, sau đó tự xóa để xóa dấu vết.

![Chuỗi API quan trọng](https://raw.githubusercontent.com/AFatc4t/aa/main/image-10.png)

Vì payload cuối chưa được giải mã nên chưa thể xác định chính xác loại mã độc sẽ được triển khai, nhưng pattern này thường xuất hiện trong chiến dịch APT hoặc crimeware nhằm nạp stealer/RAT ở giai đoạn sau. Tổ chức cần triển khai phân tích động trong sandbox cách ly để cưỡng bức loader giải mã payload thứ cấp, đồng thời giám sát log hệ thống cho các sự kiện Named Pipe/file mapping bất thường. Bổ sung memory scanning và hook monitoring trên endpoint sẽ giúp chặn các hành vi nạp mã tương tự trong tương lai.

Bên cạnh đó, nên thu thập full memory dump sau khi chạy mẫu để trích xuất shellcode, từ đó xác định MITRE ATT&CK technique (ví dụ T1055 - Process Injection, T1027 - Obfuscated/Compressed Files). Nếu phát hiện loader cố gắng self-delete, cần sao lưu bản gốc ngay khi phát hiện để tránh mất chứng cứ. Cũng nên kiểm tra hệ thống tệp xem có tạo nhật ký tạm thời hoặc file staging dưới %TEMP%/RandomName%.tmp. Tại tầng phòng thủ, đề xuất triển khai EDR với khả năng phát hiện API call sequence bất thường và bật tính năng Controlled Folder Access để ngăn loader ghi vào thư mục hệ thống.

---

**Kết luận chung cuối cùng**

CFBAFBFIEH.exe là HackTool kích hoạt bản quyền cần loại bỏ và kiểm soát chặt cơ chế licensing; các chỉ báo liên quan tới Defender exclusion và Scheduled Tasks cần được xử lý triệt để.

java.exe thực chất là miner XMRig được ngụy trang bằng tên tiến trình phổ biến, khiến tài nguyên hệ thống bị chiếm dụng và cần bị chặn cả file, persistence lẫn lưu lượng tới pool đào để tránh thất thoát chi phí vận hành.


FIDAFIEBFC.exe hoạt động như loader obfuscate, đòi hỏi phân tích sâu hơn để nhận diện payload cuối và triển khai biện pháp giám sát memory chủ động; cần chuẩn bị quy trình sandbox, memory forensics và hunting pipe bất thường cho giai đoạn theo dõi sau xử lý.

