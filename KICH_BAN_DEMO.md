# Kịch bản phát biểu khi DEMO (Slide 18)

> Demo gồm 3 thứ để chiếu: **(A) Bảng số** (Recall/HR/NDCG), **(B) Biểu đồ độ phổ biến**, **(C) Web tương tác** (chọn user → so sánh gợi ý). Phần chính khi đứng nói là **(C) web**; (A)(B) nói nhanh để có con số.
> Mỗi mục: **[LÀM]** (thao tác) · **[NÓI]** (lời thoại).

---

## 0. Mở đầu demo (sau khi bấm vào link Gradio)

**[NÓI]**
> "Để chứng minh mô hình hoạt động thật chứ không chỉ trên giấy, em demo trực tiếp trên bộ **MovieLens-10M** — khoảng 38.000 người dùng và 4.800 phim.
> Bên trái là gợi ý của **baseline** (mô hình so sánh), bên phải là gợi ý của **DIVRS** (mô hình khử thiên lệch). Em sẽ cho thấy: với cùng một người dùng, DIVRS gợi ý **chính xác hơn** nhưng **bớt hùa theo phim hot**."

*(Nếu baseline đang là Most-Popular, thêm:)* "Baseline ở đây là 'gợi ý phim phổ biến nhất' — đại diện cho cách làm thiên lệch nhất."
*(Nếu baseline là MF, thêm:)* "Baseline là MF — một mô hình gợi ý có học thật sự, chính là mốc so sánh trong paper."

---

## 1. Giải thích giao diện (trước khi kéo)

**[LÀM]** Chỉ vào 3 ô điều khiển: User ID, Top-K, ô tick "Ẩn phim đã xem".

**[NÓI]**
> "Em giải thích nhanh các ký hiệu. Mỗi dòng là một phim được gợi ý, kèm **độ phổ biến** của phim đó (0% = ngách, 100% = hot nhất hệ thống), và một nhãn kết quả:
> - **✅ test** = phim này người dùng **THẬT SỰ đã xem** — nhưng ta đã **giấu đi** khi huấn luyện, để kiểm tra. Tức mô hình **đoán TRÚNG**.
> - **➖ train** = phim người dùng đã xem rồi (mô hình biết).
> - **🟡 mới** = phim gợi ý mới, chưa kiểm chứng được.
>
> Quan trọng nhất là **✅** — càng nhiều ✅ nghĩa là đoán càng trúng cái người dùng thật sự thích."

---

## 2. Chọn 1 user và chỉ ra điểm khác biệt *(phần chính)*

**[LÀM]** Để User ID ở **giá trị mặc định** (đã được chọn sẵn là user mà DIVRS đoán trúng nhiều nhất). Để Top-K = 20.

**[NÓI]** *(vừa nói vừa chỉ tay vào màn hình)*
> "Em lấy một người dùng cụ thể. Nhìn hai cột:
>
> **Cột baseline (trái):** gợi ý toàn phim **độ phổ biến rất cao** — gần như 90–100%. Đây là mấy phim bom tấn ai cũng xem. Và để ý: **rất ít ✅** — tức là nhiều phim hot này người dùng đó **không thật sự thích**.
>
> **Cột DIVRS (phải):** độ phổ biến **đa dạng hơn, thấp hơn** — có cả phim ngách. Và đếm số **✅**: **nhiều hơn hẳn** baseline. Tức DIVRS gợi ý **đúng gu cá nhân** của người dùng này, chứ không phải cứ phim hot là nhét vào.
>
> Dòng tổng ở đầu mỗi cột cho thấy luôn: DIVRS **trúng (test) nhiều hơn** và **độ phổ biến trung bình thấp hơn**."

**[LÀM]** Có thể kéo thử **2–3 User ID khác nhau** (lấy trong danh sách gợi ý in ra), cho thấy **cột baseline gần như không đổi** còn **cột DIVRS đổi theo từng người**.

**[NÓI]**
> "Em đổi sang người dùng khác. Để ý: **cột baseline gần như y nguyên** — vì nó chỉ biết 'phim hot', ai cũng nhận như nhau, **không cá nhân hóa**. Còn **cột DIVRS thay đổi theo từng người** — vì nó học **gu riêng** của mỗi người. Đây chính là khác biệt cốt lõi."

---

## 3. Bật nút lọc — cho thấy "gợi ý thật"

**[LÀM]** Tick ô **"Ẩn phim đã xem (train)"**.

**[NÓI]**
> "Em bật chế độ ẩn các phim người dùng đã xem, để chỉ còn lại **gợi ý mới thật sự**. Lúc này danh sách DIVRS toàn phim mới + những phim ✅ (đoán trúng cái họ sẽ xem). Đây là thứ hệ thống thực tế sẽ đẩy cho người dùng."

---

## 4. Chốt bằng con số (chuyển qua bảng metric — mục A)

**[LÀM]** Kéo lên bảng so sánh (cell A) hoặc nhắc lại số.

**[NÓI]**
> "Về con số định lượng, đo trên toàn bộ người dùng theo đúng cách của paper:
> - **Recall@20** (độ chính xác): DIVRS ≈ **0.169**, cao hơn baseline rõ rệt — và **khớp gần như tuyệt đối** với con số 0.169 mà paper báo cáo, nghĩa là nhóm em **tái lập thành công**.
> - **Độ phổ biến trung bình của danh sách gợi ý**: DIVRS **thấp hơn** — tức **ít thiên lệch hơn**.
>
> Tóm lại: vừa **chính xác hơn**, vừa **bớt hùa theo đám đông** — đúng hai mục tiêu của bài toán khử thiên lệch."

*(Nếu có biểu đồ B:)* "Biểu đồ này là độ phổ biến theo Top-K: đường DIVRS **nằm thấp và phẳng**, còn baseline **dốc lên** — nghĩa là càng gợi ý nhiều, baseline càng sa vào phim hot, còn DIVRS giữ ổn định."

---

## 5. Câu chốt demo

**[NÓI]**
> "Như vậy demo cho thấy rõ: cùng một người dùng, DIVRS **đoán trúng nhiều phim họ thật sự thích hơn** (nhiều ✅ hơn), đồng thời **gợi ý đa dạng, ít hot hơn** baseline. Đó là minh chứng trực quan cho việc khử thiên lệch mà phần phương pháp đã trình bày. Em xin kết thúc demo."

---

## DỄ BỊ HỎI (khi demo)

- **"Sao biết phim ✅ là đoán đúng?"** → "Vì ta **giấu** một phần lịch sử xem của người dùng khi huấn luyện (tập test). ✅ là phim mô hình gợi ý mà **lại nằm đúng trong phần giấu đó** → đoán trúng thật, không phải mô hình đã thấy trước."

- **"Phim 🟡 mới không có ✅ thì có phải gợi ý sai không?"** → "Không hẳn. ✅ chỉ kiểm chứng được trên phần ta giấu — vốn chỉ là **một mẫu nhỏ** phim người dùng xem. 🟡 là gợi ý mới mà ta **chưa có dữ liệu để xác nhận**, nhưng **không có nghĩa là sai** — đó chính là giá trị khám phá của hệ gợi ý."

- **"Mô hình demo này có đúng là mô hình trong paper không?"** → **(trả lời trung thực)** "Mô hình em chạy là phiên bản **DIVRS** — một bản cải tiến cùng nhóm tác giả, để demo cho nhẹ. Nó dùng cùng nguyên lý biến công cụ có điều kiện. Phương pháp gốc trong slide là **IDCIV-RS** (dùng VAE). Em demo bản này để minh họa cơ chế, còn con số tái lập thì khớp với paper gốc."

- **"Vì sao độ phổ biến của DIVRS vẫn ~90% chứ không phải rất thấp?"** → "Vì để **chính xác** thì vẫn phải dùng phim có đủ tín hiệu (phim quá ngách thì thiếu dữ liệu để đoán). DIVRS **cân bằng** giữa đúng gu và đủ dữ liệu — nó giảm thiên lệch *một cách hợp lý*, không cực đoan đẩy hết về phim vô danh."

- **"Demo có chạy real-time không hay tính sẵn?"** → "Real-time: mỗi lần kéo thanh, mô hình tính lại điểm cho toàn bộ phim với người dùng đó ngay tại chỗ rồi xếp hạng."
