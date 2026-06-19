# Kịch bản thuyết trình — Phần Phương pháp IDCIV-RS (Slide 7–12)

> Chỉ gồm phần khó nhất: **Mục 3 — Phương pháp** (slide 7 đến 12 trong `DA_Advance.pptx`).
> Mỗi slide có 4 mục: **[TRÊN SLIDE]** (đang hiện gì) · **[ĐỌC]** (câu nên đọc to) · **[NÓI]** (lời thoại giải thích, nói bằng lời thường) · **[DỄ BỊ HỎI]** (câu hỏi có thể gặp + cách trả lời).

> **Mẹo chung:** đừng đọc thuật ngữ khô khan. Trước mỗi slide khó (11, 12) hãy "mồi" bằng **ví dụ đời thường** rồi mới vào toán. Hai ví dụ nên thuộc: *kem & chết đuối* (tương quan ≠ nhân quả) và *học & nhà gần thư viện* (biến công cụ).

---

## SLIDE 7 — "3.1 Quy trình ba bước của IDCIV-RS"

**[TRÊN SLIDE]** Tiêu đề "Phương pháp IDCIV-RS" + sơ đồ quy trình 3 bước.

**[ĐỌC]** "Phương pháp IDCIV-RS gồm ba bước chính."

**[NÓI]**
> "Trước khi đi vào toán, em nhắc lại bài toán ở phần trước: một cú click của người dùng **không thuần là sở thích** — nó bị trộn với nhiễu như độ phổ biến, hiệu ứng đám đông. Mục tiêu của IDCIV-RS là **tách phần sở thích thật ra khỏi phần nhiễu**.
>
> Quy trình gồm 3 bước:
> - **Bước 1 — Mã hóa:** dùng một mô hình gợi ý nền quen thuộc (MF hoặc LightGCN) để biến mỗi cặp (người dùng, sản phẩm) thành một vector đặc trưng, gọi là **W**. Lúc này W còn 'bẩn' — lẫn cả sở thích lẫn nhiễu.
> - **Bước 2 — Tách tín hiệu:** dùng một mạng VAE (module CSEM) để tách dữ liệu tương tác thành **hai thành phần**: `Zt` — phần 'sạch' đóng vai biến công cụ, và `Zc` — phần 'nhiễu/bối cảnh'.
> - **Bước 3 — Khử nhiễu & dự đoán:** dùng phần sạch `Zt` để **lọc** W, lấy ra sở thích thật, rồi dự đoán click.
>
> Ba slide tiếp theo em sẽ giải thích **vì sao** việc tách này là hợp lệ về mặt nhân quả — đó là phần lõi của paper."

**[DỄ BỊ HỎI]** *"Vì sao cần tới nhân quả, machine learning thường không làm được à?"* → "Vì ML thường chỉ học thuộc tương quan trong data, mà data đã bị nhiễu bóp méo; nó không có công cụ để phân biệt 'click vì thích' với 'click vì bị đẩy'. Nhân quả cung cấp đúng bộ luật để tách hai cái đó."

---

## SLIDE 8 — "3.2 Sơ đồ nhân quả — biến W, Y, Uc"

**[TRÊN SLIDE]** Sơ đồ DAG + giải nghĩa 3 biến: W (Treatment), Y (Outcome), Uc (Latent Confounder).

**[ĐỌC]** "Sơ đồ nhân quả mô tả quan hệ giữa các biến. Ba biến đầu tiên: W, Y và Uc."

**[NÓI]**
> "Em giới thiệu các biến trên sơ đồ, đây là ngôn ngữ của suy luận nhân quả:
> - **W — biến can thiệp (treatment):** cái mà hệ thống 'tác động', cụ thể là việc **hiển thị/gợi ý một sản phẩm** cho người dùng. Về kỹ thuật, W là embedding của cặp (user, item).
> - **Y — biến kết quả (outcome):** **phản hồi của người dùng** — click, xem, mua.
> - **Uc — biến nhiễu ẩn (latent confounder):** các yếu tố **không quan sát được** như tâm trạng nhất thời, xu hướng ảo, hiệu ứng đám đông.
>
> Điểm mấu chốt: **Uc tác động đồng thời lên CẢ W lẫn Y** — thể hiện bằng hai mũi tên nét đứt từ Uc sang W và sang Y. Chính vì nó kéo cả hai đầu nên nó tạo ra mối tương quan giả mà ta phải gỡ."

**[DỄ BỊ HỎI]** *"Cho ví dụ Uc cụ thể?"* → "Phim đang là trend (hiệu ứng đám đông): trend vừa khiến hệ thống đẩy phim đó lên (tác động W), vừa khiến người dùng tò mò bấm vào (tác động Y) — dù họ không thật sự thích thể loại đó."

---

## SLIDE 9 — "3.2 Sơ đồ nhân quả — biến X, Zt, Zc"

**[TRÊN SLIDE]** Giải nghĩa: X (biến tiền can thiệp, khung nét đứt), Zt (CIV), Zc (Conditioning Set).

**[ĐỌC]** "Tiếp theo là ba biến mà IDCIV-RS làm việc: X, Zt và Zc."

**[NÓI]**
> "- **X — biến tiền can thiệp:** là tập đặc trưng **quan sát được TRƯỚC khi** việc hiển thị xảy ra — cụ thể là **lịch sử tương tác cũ** của người dùng. Đây là nguyên liệu đầu vào.
> - Từ X, mô hình **tự động trích xuất ra hai thành phần** `Zt` và `Zc`:
>   - **Zt — Biến công cụ có điều kiện (Conditional IV, viết tắt CIV):** đây là 'đòn bẩy sạch' mà em sẽ giải thích ở slide sau.
>   - **Zc — Tập điều kiện kiểm soát (Conditioning Set):** phần đi kèm để 'chặn' các đường nhiễu.
>
> Điểm đột phá của paper nằm ở đây: thay vì đi tìm một biến công cụ từ bên ngoài (như log tìm kiếm của người dùng — thứ thường không có sẵn), nó **tự nặn `Zt` và `Zc` ra từ chính dữ liệu X**."

**[DỄ BỊ HỎI]** *"Biến công cụ là gì?"* → Kể nhanh ví dụ thư viện: "Muốn biết 'học nhiều có làm điểm cao không' nhưng bị nhiễu bởi 'độ giỏi bẩm sinh'. Ta tìm thứ đẩy người ta học nhiều mà không liên quan độ giỏi — ví dụ 'nhà gần thư viện'. Đó là biến công cụ. `Zt` đóng vai trò tương tự cho bài toán này."

---

## SLIDE 10 — "Cơ chế chặn đường Backdoor bằng CIV"

**[TRÊN SLIDE]** Zt (CIV), Zc (Conditioning Set), và ý "Chặn backdoor hoàn hảo".

**[ĐỌC]** "Cơ chế cốt lõi: dùng biến công cụ có điều kiện để chặn đường truyền backdoor."

**[NÓI]**
> "Slide này nêu cơ chế, slide sau sẽ phân tích vì sao nó đúng.
> - `Zt` là biến công cụ học từ dữ liệu tương tác thực tế X.
> - `Zc` là **tập điều kiện đi kèm**, nhiệm vụ của nó là **chặn các liên kết không hợp lệ** (các đường nhiễu).
> - Cơ chế: **khi ta 'kiểm soát'/cố định dựa trên Zc** (gọi là *điều kiện hóa* — conditioning), thì **mọi đường truyền 'cửa sau' (backdoor) phi nhân quả từ Zt tới click Y đều bị chặn**. Nhờ vậy ta **cô lập được tác động thuần túy của việc hiển thị W lên Y**.
>
> Nói nôm na: `Zc` đóng vai **tấm chắn** — nó hứng hết các đường nhiễu để `Zt` còn lại 'sạch', chỉ ảnh hưởng kết quả thông qua W."

**[DỄ BỊ HỎI]** *"'Đường backdoor' là gì?"* → "Là đường đi vòng phi nhân quả nối hai biến qua một yếu tố thứ ba, làm chúng tương quan giả. Slide 11 em sẽ vẽ rõ đường đó."

---

## SLIDE 11 — "Phân tích cơ chế: vấn đề backdoor + rào cản Uc–Zc"

**[TRÊN SLIDE]** Hai ý: (1) Vấn đề đường backdoor `W ← Uc → Y`; (2) đường nét đứt `Uc → Zc`.

**[ĐỌC]** "Tại sao mô hình loại bỏ được thiên lệch? Bắt đầu từ vấn đề của đường truyền sai số."

**[NÓI]** *(đây là slide khó — nói chậm, mồi ví dụ)*
> "Em mồi bằng ví dụ: số liệu cho thấy **ngày nào bán nhiều kem thì cũng nhiều người chết đuối**. Không phải kem gây chết đuối — mà **trời nóng** kéo cả hai. 'Trời nóng' là yếu tố thứ ba ẩn. Bài toán của ta y hệt vậy.
>
> **Vấn đề 1 — Đường backdoor:** nhiễu ẩn `Uc` tạo ra đường đi vòng **`W ← Uc → Y`**. Đường này khiến mô hình ML truyền thống **ngộ nhận** rằng người dùng click vì thích sản phẩm (`W → Y`), trong khi thực ra họ click vì bị `Uc` tác động. Đây chính là 'kem & chết đuối' — tương quan giả.
>
> **Vấn đề 2 — Vì sao không lấy biến công cụ trực tiếp được:** trên sơ đồ có **đường nét đứt cong từ `Uc` sang `Zc`**. Nó nói rằng **nhiễu ẩn vẫn rò rỉ vào dữ liệu đầu vào**. Hệ quả: ta **không thể tìm được một biến công cụ thuần túy, hoàn toàn sạch khỏi nhiễu** nếu chỉ lấy trực tiếp từ hệ thống. → Đây là lý do phải dùng tới **biến công cụ CÓ ĐIỀU KIỆN** (slide 12), thay vì biến công cụ thường."

**[DỄ BỊ HỎI]** *"Vậy đường Uc→Zc là điểm yếu hay điểm mạnh?"* → "Vừa là rào cản vừa là chìa khóa: vì `Zc` có dính `Uc` nên khi ta điều kiện hóa trên `Zc`, ta **gián tiếp kiểm soát luôn `Uc`** → chặn được nhiễu. Nếu `Zc` hoàn toàn tách rời `Uc` thì nó vô dụng."

---

## SLIDE 12 — "Giải pháp Conditioning + Kết luận" *(slide quan trọng nhất)*

**[TRÊN SLIDE]** Cơ chế điều kiện hóa: `Zc` hấp thụ nhiễu, `Zt` thành CIV hợp lệ (`Zt→W→Y`), và phần Kết luận.

**[ĐỌC]** "Giải pháp của IDCIV-RS: cơ chế điều kiện hóa, tách dữ liệu X thành hai vai trò."

**[NÓI]** *(điểm nhấn của cả phần phương pháp)*
> "Giải pháp: mô hình **tách dữ liệu X thành hai phần có vai trò khác nhau**:
>
> **(1) `Zc` gánh chịu và hấp thụ nhiễu.** Trên sơ đồ, `Zc` có mũi tên tới cả `W` và `Y`, nên nó đóng vai **biến kiểm soát cấu trúc nhiễu**. **Khi mô hình điều kiện hóa dựa trên `Zc`, đường truyền ảnh hưởng từ nhiễu ẩn `Uc` bị chặn đứng.**
>
> **(2) `Zt` trở thành biến công cụ sạch (Valid CIV).** Sau khi `Zc` đã khóa đường nhiễu, `Zt` chỉ còn tác động tới kết quả `Y` **qua một con đường DUY NHẤT: đi qua biến can thiệp W**, tức `Zt → W → Y`. Đúng định nghĩa một biến công cụ hợp lệ.
>
> **Kết luận:** nhờ ép dữ liệu tuân theo cấu trúc nhân quả này, IDCIV-RS **cô lập và đo được tác động thuần túy `W → Y`** (sở thích thật), **loại bỏ tương quan giả do `Uc`** — và quan trọng là **không cần bất kỳ dữ liệu ngoại sinh nào từ bên ngoài** (không cần log tìm kiếm). Đó là điểm vượt trội so với các phương pháp biến công cụ thế hệ trước."

**[DỄ BỊ HỎI]**
- *"'Điều kiện hóa trên Zc chặn nhiễu' — vì sao chặn được?"* → "Vì `Zc` nằm trên đường nối nhiễu tới kết quả (và có dính `Uc`). Cố định `Zc` lại giống như 'bịt' điểm trung chuyển đó → tín hiệu nhiễu không lan qua được. Trong lý thuyết đồ thị, điều kiện hóa trên biến giữa của một đường rẽ nhánh sẽ chặn đường đó."
- *"Làm sao chắc VAE tách đúng sạch/nhiễu?"* → **(trả lời trung thực)** "Đây là phần mô hình *kỳ vọng* đạt được qua thiết kế VAE, **không có bảo chứng tuyệt đối** — disentanglement của VAE nói chung khó định danh. Đây là một giả định/điểm yếu lý thuyết của phương pháp; bù lại thực nghiệm cho thấy nó hoạt động tốt."
- *"Chứng minh hình thức `Zt` là CIV ở đâu?"* → "Trong paper dùng d-separation trên đồ thị can thiệp `G_W` (xóa mũi tên ra khỏi W): chứng minh `Zt` liên quan W, và mọi đường khác từ `Zt` tới Y đều bị chặn (qua collider tại W và qua điều kiện trên `Zc`). Em có thể trình bày nếu thầy cần."

---

## Câu chốt chuyển sang phần Dữ liệu (sau slide 12)
> "Như vậy phần phương pháp đã xong: IDCIV-RS tự học biến công cụ có điều kiện từ data để gỡ nhiễu mà không cần dữ liệu ngoài. Tiếp theo em trình bày dữ liệu và thực nghiệm để kiểm chứng nó thật sự hoạt động."

---

## Phụ lục — Thứ tự ý để KHÔNG bị rối khi nói phần nhân quả (slide 11–12)
1. Có nhiễu ẩn `Uc` → tạo đường cửa sau `W ← Uc → Y` → tương quan giả (ví dụ kem/chết đuối).
2. Muốn gỡ thì cần "biến công cụ sạch", nhưng `Uc` rò rỉ vào data (`Uc⋯Zc`) → **không có** biến công cụ thuần túy.
3. Mẹo: tách data thành `Zt` (đòn bẩy) + `Zc` (tấm chắn nhiễu).
4. **Điều kiện hóa trên `Zc`** → chặn đường nhiễu → `Zt` thành **CIV hợp lệ** (chỉ tác động Y qua W).
5. ⇒ Đo được `W → Y` sạch, không cần dữ liệu ngoài.
