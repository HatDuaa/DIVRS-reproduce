# Hiểu phương pháp IDCIV-RS bằng ngôn ngữ đời thường

> Bản này giải thích **ý tưởng cốt lõi** của paper, đi từ con số 0, **hạn chế tối đa thuật ngữ**. Mỗi khi buộc phải dùng một từ chuyên môn, sẽ có giải thích ngay bằng lời thường. Đọc xong bản này rồi mới sang bản kỹ thuật (`PHUONG_PHAP_IDCIV-RS.md`) sẽ dễ hơn nhiều.

---

## Phần 1 — Câu chuyện: Netflix đang đoán sai

Tưởng tượng mày làm hệ gợi ý phim cho Netflix. Mục tiêu: **đoán phim mà user thật sự thích** để gợi ý.

Dữ liệu mày có chỉ là **lịch sử bấm xem**: user nào đã bấm vào phim nào.

Suy nghĩ tự nhiên: *"User bấm xem phim A → chắc user thích phim A. Cứ tìm phim giống A mà gợi ý."*

Nghe hợp lý. **Nhưng đây là cái bẫy lớn nhất của cả bài toán.**

---

## Phần 2 — Cái bẫy: "bấm xem" KHÔNG có nghĩa là "thích"

Hỏi lại: vì sao user bấm vào một phim? Có **nhiều lý do**, và phần lớn **không phải vì thích**:

1. **Vì phim đó đang hot.** Phim bom tấn được Netflix nhét lên trang chủ, banner to đùng, ai vào cũng thấy. User bấm vì **nó đập vào mắt**, không phải vì hợp gu.
2. **Vì đám đông xem.** "Cả thế giới đang xem phim này" → user xem theo cho khỏi lạc hậu, dù thể loại đó user không mê.
3. **Vì vị trí hiển thị.** Phim nằm ở ô đầu tiên được bấm nhiều hơn phim ở ô thứ 50, dù phim thứ 50 có khi hợp gu hơn.

→ Nói cách khác: **một cú bấm xem là hỗn hợp của "sở thích thật" + "bị hoàn cảnh đẩy đưa".**

Cái "hoàn cảnh đẩy đưa" (độ hot, đám đông, vị trí) — ta gọi nó là **yếu tố gây nhiễu**. Nó **ẩn**: nó không nằm trong dữ liệu (Netflix đâu có ghi "user này bấm vì phim đang hot"), nhưng nó **bóp méo** mọi cú bấm.

**Ví dụ cụ thể:** một user mê phim tài liệu lịch sử (ngách, ít người xem). Nhưng trong lịch sử bấm của họ lại đầy phim Marvel — vì Marvel lúc nào cũng hot, ai cũng bị đẩy xem. Nếu mày chỉ nhìn data, mày sẽ kết luận "user này thích Marvel" và gợi ý toàn siêu anh hùng. **Sai bét** — đó là do hype, không phải gu thật.

---

## Phần 3 — Vì sao máy học thông thường bị lừa

Các mô hình gợi ý phổ biến (Matrix Factorization, deep learning...) về bản chất làm **một việc duy nhất**: *học thuộc lịch sử bấm xem rồi bắt chước*.

Vấn đề: lịch sử bấm xem **đã bị nhiễu** (đầy hype). Máy học thuộc cả cái nhiễu đó. Kết quả:
- Phim hot vốn được bấm nhiều → máy tưởng "ai cũng thích" → **gợi ý nó cho nhiều người hơn nữa**.
- Được gợi ý nhiều hơn → lại được bấm nhiều hơn → máy lại càng chắc "phim này hay".

Đây là **vòng xoáy**: thiên lệch tự khuếch đại. Đồ hot ngày càng hot, đồ ngách ngày càng bị chôn vùi. Sở thích thật của user bị bỏ rơi.

**Gốc rễ:** máy học thường **không phân biệt được hai chuyện trông y hệt nhau trong dữ liệu:**
- (A) User bấm vì **thật sự thích** → đây là cái ta CẦN.
- (B) User bấm vì **phim hot đẩy vào mặt** → đây là cái ta muốn LOẠI.

Cả hai đều hiện ra dưới dạng "một cú bấm". Làm sao tách? Máy học thuần **không có công cụ để tách** → đây là lúc cần tới **suy luận nhân quả**.

---

## Phần 4 — "Nhân quả" thật ra là gì? (đừng sợ từ này)

Suy luận nhân quả **không phải một loại mạng neural mới**. Nó là một **cách tư duy / bộ quy tắc** để trả lời đúng một câu hỏi: 

> **"X có THẬT SỰ gây ra Y không, hay X và Y chỉ TÌNH CỜ đi cùng nhau vì một lý do thứ ba?"**

Ví dụ kinh điển (không liên quan phim, để thấy ý):
- Số liệu cho thấy: **ngày nào bán nhiều kem thì cũng nhiều người chết đuối**. Hai con số đi cùng nhau.
- Vậy **ăn kem gây chết đuối**? Tất nhiên không.
- Lý do thật: **trời nóng** (yếu tố thứ ba). Trời nóng → vừa khiến người ta ăn kem nhiều, **vừa** khiến người ta đi bơi nhiều (nên chết đuối nhiều). Kem và chết đuối chỉ **tình cờ đi cùng** vì cùng bị "trời nóng" kéo.

→ Bài học: **"đi cùng nhau" (tương quan) KHÔNG bằng "gây ra nhau" (nhân quả).** Và thủ phạm thường là một **yếu tố thứ ba ẩn** kéo cả hai.

Quay lại bài toán phim, mày sẽ thấy nó **y hệt**:
- "User bấm phim" và "phim đó hợp gu user" — trông như đi cùng nhau.
- Nhưng **độ hot** (yếu tố thứ ba) kéo cả "user bấm" lẫn "phim được show". Nên một phần "user bấm" chỉ là **tình cờ đi cùng** do hot, không phải vì hợp gu.

**Suy luận nhân quả cho ta luật để gỡ cái "tình cờ" này ra.** Đó là lý do paper phải dùng nó — chứ máy học thuần không gỡ được.

---

## Phần 5 — Mẹo cốt lõi: tìm "đòn bẩy sạch" *(đây là phần quan trọng nhất, đọc chậm)*

Làm sao gỡ ảnh hưởng của yếu tố thứ ba (độ hot) khi ta **không đo được nó**? Có một mẹo rất hay trong nhân quả. Tao kể bằng ví dụ khác cho dễ thấm:

**Câu hỏi:** "Học nhiều giờ có làm điểm cao hơn không?"

**Rắc rối:** học sinh **thông minh & chăm** thì **vừa học nhiều, vừa điểm cao**. Nên nhìn số liệu thấy "học nhiều = điểm cao", nhưng **không biết** là do *học* hay do *bản thân đứa đó vốn giỏi*. "Độ giỏi bẩm sinh" chính là yếu tố thứ ba ẩn.

**Mẹo:** tìm một thứ **đẩy người ta học nhiều hơn, NHƯNG bản thân nó chẳng liên quan gì tới độ giỏi**. Ví dụ: **nhà ở gần thư viện hay xa thư viện** (chuyện này khá ngẫu nhiên, không dính tới đứa đó giỏi hay dốt).

- Đứa ở gần thư viện → tiện → học nhiều hơn.
- Đứa ở xa → lười ra → học ít hơn.
- Cái khác biệt "học nhiều/ít" mà **do khoảng cách thư viện** gây ra là phần **SẠCH** — vì khoảng cách thư viện không dính tới độ giỏi.

→ Giờ chỉ cần nhìn: *trong nhóm "ở gần thư viện học nhiều", điểm có cao hơn nhóm "ở xa học ít" không?* Nếu có → **học thật sự làm điểm cao**, và ta đo được con số sạch, không bị "độ giỏi" làm nhiễu.

Cái thứ đóng vai "**đòn bẩy sạch**" đó (nhà gần thư viện) — trong nhân quả gọi là **biến công cụ** (instrumental variable). Đặc điểm của nó:
1. Nó **lay chuyển được nguyên nhân** ta quan tâm (đẩy người ta học nhiều/ít).
2. Nó **sạch** — không dính yếu tố thứ ba (không liên quan độ giỏi), và **chỉ ảnh hưởng kết quả (điểm) thông qua nguyên nhân (học)**, không có đường tắt nào khác.

**Đây là toàn bộ tinh thần của "biến công cụ". Nắm cái này là nắm 80% paper.**

---

## Phần 6 — Áp "đòn bẩy sạch" vào bài toán phim

- **Nguyên nhân** ta quan tâm: cặp (user, phim) hợp nhau cỡ nào → quyết định bấm.
- **Kết quả:** user có bấm không.
- **Yếu tố thứ ba ẩn:** độ hot / đám đông.

**Đòn bẩy sạch lý tưởng** = thứ khiến user tương tác với phim **vì lý do thật của họ, không phải vì hype**. Ví dụ điển hình: **user TỰ TÌM KIẾM (search) ra phim đó rồi mới bấm.** Hành động search thể hiện **ý định thật** — họ chủ động muốn, không phải bị nhét vào mặt.

Nếu Netflix có log search, ta dùng nó làm đòn bẩy sạch: "phần bấm xem mà **do user tự search** ra" = tín hiệu sạch của gu thật → dùng nó để học sở thích, bỏ qua phần bấm do hype.

**Nhưng** — và đây là vấn đề — **không phải hệ nào cũng có log search sạch sẽ như vậy.** Vậy lấy "đòn bẩy sạch" ở đâu ra?

---

## Phần 7 — Đột phá của paper: TỰ NẶN đòn bẩy sạch từ dữ liệu

Đây là đóng góp chính. Thay vì đi xin "đòn bẩy sạch" từ bên ngoài (log search), paper **tự tạo nó ra từ chính lịch sử tương tác**, bằng một mạng neural tên là **VAE**.

**VAE là gì (nói cho dễ):** hãy hình dung nó như một cái **máy phân loại tín hiệu**. Mày đổ vào một mớ dữ liệu lộn xộn, nó học cách **nén lại** rồi **tách ra thành các thành phần** có ý nghĩa, sao cho từ các thành phần đó có thể **dựng lại** dữ liệu gốc. Giống như đưa một bản nhạc thu lẫn lộn, nó tách ra "giọng hát" và "nhạc nền" riêng.

Paper bắt VAE **tách lịch sử tương tác thành 2 phần**:
- **Phần A — "ý định sạch"** (ký hiệu `Z_t`): đóng vai **đòn bẩy sạch** (như cái search ẩn). Phần này phản ánh **lý do thật** khiến user tương tác.
- **Phần B — "bối cảnh nhiễu"** (ký hiệu `Z_c`): hứng phần **hype / đám đông / hoàn cảnh**. Phần này gom hết cái bẩn.

Mục tiêu khi huấn luyện VAE: làm cho **hai phần này tách bạch nhau** — phần A càng sạch (chỉ chứa ý định), phần B càng gom trọn nhiễu.

> ⚠️ **Điểm cần thành thật (để khỏi bị bắt bẻ khi thuyết trình):** việc VAE tách "sạch" ra "sạch" và "nhiễu" ra "nhiễu" là điều paper **mong muốn và thiết kế hướng tới**, chứ **không có gì đảm bảo tuyệt đối** nó tách đúng. Đây là điểm yếu lý thuyết của hướng tiếp cận này.

---

## Phần 8 — Dùng phần sạch để lấy ra "sở thích thật"

Có "đòn bẩy sạch" (`Z_t`) rồi, dùng nó thế nào?

Nhớ lại embedding gốc của cặp (user, phim) — gọi là `W` — đang **lẫn lộn** cả sở thích thật lẫn nhiễu hype.

Paper làm một phép **"lọc"**: chiếu `W` lên phần sạch `Z_t`, để giữ lại **chỉ phần của `W` mà "đòn bẩy sạch" giải thích được**. Hình dung như **lọc cà phê**: đổ hỗn hợp qua phin, giữ lại nước cốt (sở thích thật), bỏ bã (nhiễu).

Kết quả là một embedding mới, gọi là `Ŵ` ("W mũ") = **sở thích thật, đã gột sạch hype**. Đây chính là cái ta muốn để gợi ý đúng gu.

*(Về mặt toán, phép "lọc" này là phép chiếu bình phương tối thiểu — nhưng mày không cần nhớ công thức, chỉ cần hiểu "lọc lấy phần sạch".)*

---

## Phần 9 — Bốn bước của paper, kể lại bằng lời thường

1. **Mã hóa:** dùng mô hình nền (MF/LightGCN — mấy mô hình gợi ý quen thuộc) biến mỗi cặp (user, phim) thành một dãy số (embedding) `W`. Lúc này `W` còn bẩn (lẫn nhiễu).

2. **Tách (bằng VAE):** đổ lịch sử tương tác vào VAE, tách ra **phần sạch `Z_t`** (đòn bẩy) và **phần nhiễu `Z_c`** (bối cảnh).

3. **Lọc:** dùng phần sạch `Z_t` để lọc `W`, lấy ra **`Ŵ` = sở thích thật**. Sau đó **trộn lại một chút phần nhiễu `Z_c`** (xem Phần 10 vì sao) để ra embedding cuối `W^re`.

4. **Dự đoán:** dùng `W^re` để chấm điểm phim nào hợp với user, rồi gợi ý — huấn luyện bằng cách ép "phim user thật sự thích phải được điểm cao hơn phim user không thích" (đây là **BPR**, một cách huấn luyện gợi ý phổ biến).

Toàn bộ học **một lượt cùng nhau**: vừa tách sạch/nhiễu, vừa dự đoán click cho tốt.

---

## Phần 10 — Khoan, vì sao lọc sạch rồi lại trộn nhiễu vào? (câu dễ hỏi nhất)

Thoạt nghe vô lý: gột sạch nhiễu xong sao lại cộng `Z_c` (nhiễu) trở lại ở bước 3?

Lý do: ta muốn **gợi ý đúng gu**, nhưng vẫn phải **dự đoán đúng trong thế giới thực — nơi hype vẫn tồn tại**. Nếu hoàn toàn phớt lờ bối cảnh, mô hình sẽ dự đoán click kém (vì thực tế người ta vẫn bị hype chi phối). 

Nên paper **cân bằng**: phần lớn là **sở thích thật `Ŵ`**, pha thêm chút **bối cảnh `Z_c`** để "hiểu" môi trường. Tỉ lệ pha do một nút vặn (tham số `α`) điều chỉnh — paper để 80% sạch + 20% bối cảnh.

→ Trực giác: "**biết gu thật của user, nhưng cũng hiểu họ đang ở trong môi trường đầy hype**" thì dự đoán mới vừa đúng gu vừa sát thực tế.

---

## Phần 11 — Bức tranh tổng & một câu để nhớ

Ghép lại toàn bộ:

```
Lịch sử bấm xem (BẨN: lẫn sở thích + hype)
        │
        ▼  VAE tách làm 2
   ┌────────────┬─────────────┐
   │ Z_t: ý định│ Z_c: bối     │
   │ sạch (đòn   │ cảnh nhiễu   │
   │ bẩy)        │ (hype)       │
   └─────┬──────┴──────┬───────┘
         │ lọc W        │ pha thêm chút
         ▼              ▼
   Ŵ = SỞ THÍCH THẬT  →  W^re = 80% thật + 20% bối cảnh
                         │
                         ▼
                    Gợi ý phim (đúng gu, bớt hype)
```

> **MỘT CÂU ĐỂ NHỚ:**
> *Cú bấm của user bị trộn giữa "thích thật" và "bị hype đẩy". Paper dùng AI (VAE) để **tách** hai cái đó ra, rồi — theo luật "đòn bẩy sạch" của suy luận nhân quả — dùng phần **ý định sạch** để lọc lấy **sở thích thật**, nhờ vậy gợi ý đúng gu thay vì hùa theo đồ hot.*

---

## Phần 12 — Vậy "nhân quả" nằm ở đâu, "AI" nằm ở đâu?

Để trả lời thắc mắc "nó không giống bài AI thuần":

| Thành phần | Thuộc về | Lo việc gì |
|---|---|---|
| MF / LightGCN (mã hóa) | **AI quen thuộc** | Biến user/phim thành dãy số |
| VAE (tách sạch/nhiễu) | **AI quen thuộc** | Cái máy phân loại tín hiệu |
| BPR (huấn luyện) | **AI quen thuộc** | Ép gợi ý đúng lên top |
| **Luật "đòn bẩy sạch" (biến công cụ)** | **Suy luận nhân quả** | **Cho biết: vì sao tách như vậy là HỢP LỆ, phần Z_t mới đáng tin để gỡ nhiễu** |

→ Paper = **bộ khung AI bình thường** + **một lớp lý luận nhân quả ở trên** để đảm bảo việc "tách & lọc" là đúng đắn chứ không phải làm bừa. Phần nhân quả không thêm mạng neural nào — nó thêm **lý do để tin** rằng cách làm này gỡ được thiên lệch.

Mấy từ đáng sợ trong tài liệu kỹ thuật (`collider`, `d-separation`, `G_W`...) chỉ là **công cụ toán học để CHỨNG MINH** rằng "phần sạch `Z_t` đúng là một đòn bẩy hợp lệ". **Hiểu Phần 1–11 là đã nắm hồn paper; phần chứng minh để dành khi cần phản biện sâu.**

---

## Phần 13 — Mấy câu hỏi đào sâu (gom từ thảo luận)

### 13.1. VAE tách "sạch / nhiễu" bằng cách nào? (chỗ mơ hồ nhất)

Một hiểu lầm thường gặp: *"VAE nhìn vào độ phổ biến của phim, phim nào ngách mà vẫn được bấm thì gắn nhãn sạch."* — **Không phải vậy.**

VAE **KHÔNG** được cho biết phim nào hot, phim nào ngách. Nó làm việc **không giám sát (unsupervised)**:
- Mày đổ **lịch sử tương tác** vào, và bảo: *"Nén dữ liệu này vào **2 cái hộp tách biệt**, sao cho từ 2 hộp dựng lại được dữ liệu gốc."*
- VAE **tự mò** ra cách chia. Không ai nói hộp nào là "nhiễu".

**Vì sao nó có xu hướng chia đúng — nhờ cấu trúc dữ liệu:**
- Phim hot nằm trong lịch sử của **rất nhiều user** → là **mẫu hình chung, lặp đi lặp lại** → VAE dồn vào **hộp nhiễu** (`Z_c`).
- Phim hợp gu thì **riêng từng user** → là **mẫu hình đặc thù** → VAE dồn vào **hộp sạch** (`Z_t`).

→ Hình dung: VAE giống cái máy tách "**cái-ai-cũng-có**" (hype) khỏi "**cái-riêng-của-từng-người**" (gu). 

> ⚠️ Nhưng đây là **xu hướng/kỳ vọng, KHÔNG đảm bảo**. Không có gì ép VAE phải đặt đúng popularity vào hộp nhiễu — đây là điểm yếu lý thuyết của hướng này.

*(Trực giác "phim ngách mà vẫn bấm = tín hiệu sạch" của mày **đúng tinh thần** — vì phim ngách không có hype, bấm nó nhiều khả năng vì thích thật. Nhưng đó là cách **ta hiểu vì sao method hợp lý**, không phải luật mà VAE thực thi. Độ phổ biến được dùng tường minh ở chỗ KHÁC — xem 13.3.)*

### 13.2. "Mẫu âm" (negative sample) là gì?

Khi dạy mô hình gợi ý, cần **2 loại ví dụ để đối chiếu**:
- **Mẫu DƯƠNG:** cặp (user, phim) user **đã bấm** → "thích".
- **Mẫu ÂM:** cặp (user, phim) user **chưa bấm** → coi như "không thích / không quan tâm".

**Vì sao cần mẫu âm?** Mô hình học bằng cách **so sánh**: phải cho **phim thích điểm CAO hơn phim không thích**. Nếu chỉ có mẫu dương → mô hình chấm mọi phim đều cao → vô dụng. Phải có "cái xấu" để biết "cái tốt".

> Như dạy trẻ "đây là **con mèo**" (dương) thì cũng phải chỉ "đây **không phải** mèo (con chó)" (âm) → nó mới học được ranh giới.

**Rắc rối + vì sao có PNSM:** ta chỉ có data "đã bấm", **không có data "ghét"**. Nên mẫu âm phải **bịa ra** bằng cách lấy phim user chưa bấm. Nguy hiểm: phim "chưa bấm" có khi là phim user **sẽ thích** (chỉ chưa thấy) → lấy nhầm làm mẫu âm thì dạy sai. **PNSM** = mẹo chọn mẫu âm khôn ngoan (dựa độ phổ biến + ngưỡng) để giảm rủi ro chọn nhầm.

### 13.3. Tập "skew" (ngách) — có phải train lại lần nữa? Có trùng data gốc không?

**Không train lại.** Chỉ **1 lần train duy nhất**, dữ liệu được **trộn chung** trong cùng vòng lặp (code: hàm "blend" = trộn; mỗi epoch chạy cả data thường + skew).

**Và đúng, skew TRÙNG data gốc** — nó **không phải tương tác mới**. Nó lấy từ **chính đống tương tác cũ**, chỉ khác **CÁCH lấy mẫu (tần suất)**:
- **Data thường:** lấy theo phân bố tự nhiên → **phim hot áp đảo** (vì hot nên nhiều lượt bấm).
- **Skew:** lấy **cùng đống đó** nhưng **cân lại** — cho phim **ÍT phổ biến xuất hiện NHIỀU hơn** (gán trọng số `1/số_lượt_xem` rồi rút mẫu theo trọng số).

→ **Cùng phim, chỉ khác "ống kính".** Giá trị không nằm ở "data mới" mà ở **đổi khẩu phần chú ý**: trộn skew vào → kéo sự chú ý của mô hình về phía phim ngách → **chống lại sự áp đảo của phim hot** khi học.

> Ví dụ: lớp ôn thi 90% câu chủ đề A (hot), 10% chủ đề B (ngách). Ôn theo tỉ lệ tự nhiên → giỏi A dốt B. "Bộ đề skew" = **cùng mấy câu đó** nhưng cho **luyện câu B nhiều hơn** → cân lại, giỏi cả hai. Vẫn câu cũ, chỉ khác mức độ luyện.

**Đây cũng chính là nơi "độ phổ biến" được dùng TƯỜNG MINH** (khác với VAE ở 13.1 vốn tự mò). Và nó khớp đúng trực giác "ưu tiên tương tác với đồ ngách = tín hiệu gu thật".

### 13.4. PNSM là gì? (cách chọn mẫu âm thông minh)

**PNSM = Popularity-based Negative Sampling with Margin** (lấy từ paper DICE). Tách từng chữ:
- **Negative Sampling** = lấy mẫu âm (phim "không thích" để đối chiếu — xem 13.2).
- **Popularity-based** = chọn mẫu âm **dựa trên độ phổ biến** (so với phim dương user đã bấm), không lấy đại ngẫu nhiên.
- **with Margin** = có **ngưỡng chênh lệch**: chỉ xét phim có độ phổ biến **chênh đủ lớn** so với phim dương (bỏ qua phim phổ biến gần gần — so sánh không rõ ràng).

**Mẹo hay nằm ở đây:** ưu tiên lấy **phim HOT HƠN phim dương** làm mẫu âm. Vì sao?

> User đã chọn phim **ÍT hot** (phim dương `p`) mà **bỏ qua** một phim **HOT hơn** (mẫu âm `n`).

Lựa chọn này là **bằng chứng MẠNH của gu thật**: nếu chỉ hùa theo đám đông, user đã bấm phim hot hơn (`n`). Nhưng họ chọn phim ngách (`p`) → **thích thật, không phải vì hot**. Dạy mô hình theo cặp "user thích `p` (ngách) hơn `n` (hot)" sẽ **bơm đúng tín hiệu sở thích thật**, gột bớt thiên lệch phổ biến.

**Ví dụ:**
- User bấm "phim tài liệu lịch sử" (ít người xem) thay vì "Avengers" (cả thế giới xem).
- PNSM lấy **Avengers làm mẫu âm**, dạy: *"điểm(phim tài liệu) phải > điểm(Avengers) cho user này."*
- → Mô hình học: user này **thật sự** thích tài liệu, không phải cứ hot là thích.

**So với lấy mẫu âm ngẫu nhiên thường:**
- *Ngẫu nhiên:* lấy đại phim chưa bấm → dễ lấy nhầm phim user sẽ thích → tín hiệu loãng.
- *PNSM:* lấy có chủ đích (phim hot hơn, chênh đủ ngưỡng) → tạo cặp đối chiếu **"ngách thắng hot"** → tín hiệu gu thật rõ hơn.

→ Tóm: PNSM = chọn mẫu âm **là phim hot hơn (chênh đủ ngưỡng)**, để mỗi lần học là một lần khẳng định "user chọn đồ hợp gu thay vì đồ hot" — đúng tinh thần khử thiên lệch.
