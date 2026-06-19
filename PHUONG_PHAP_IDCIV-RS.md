# Giải thích phương pháp IDCIV-RS (Phần 3 của paper)

> Paper gốc: *Interaction-Data-guided Conditional Instrumental Variables for Debiasing Recommender Systems* (Huang, Cheng, Liu, Li, Lu, Zhang — IJCAI 2025).
> Tài liệu này giải thích **Mục 3 (Phương pháp)** và **toàn bộ nền tảng nhân quả** đứng sau nó, theo cách dễ hiểu nhất, để phục vụ phần trình bày seminar.

---

## 0. Đặt vấn đề (bài toán là gì)

Hệ gợi ý học sở thích người dùng từ **dữ liệu hành vi** (lượt click/xem). Giả định ngầm của hầu hết mô hình (MF, NCF...): *dữ liệu hành vi phản ánh trung thực sở thích*.

**Nhưng thực tế không phải vậy.** Hành vi người dùng bị bóp méo bởi các **biến gây nhiễu ẩn (latent confounders)** — ký hiệu `U_c` — mà ta **không quan sát được** và **không nằm trong log**:
- **Độ phổ biến (popularity bias):** item bị đẩy lên nhiều → người dùng "buộc" thấy nó nhiều → click nhiều, không phải vì thích.
- **Hiệu ứng đám đông (conformity bias):** xem phim vì bạn bè xem, vì xu hướng — không phải vì gu cá nhân.
- **Vị trí hiển thị, khuyến mãi, search...**

→ Mô hình học phải **tương quan giả** (spurious correlation), gợi ý ngày càng thiên về đồ phổ biến, và **vòng lặp khuếch đại thiên lệch**. Mục tiêu của paper: **khử thiên lệch (debias)** — tách "sở thích thật" khỏi "nhiễu do `U_c`".

---

## 1. Ngôn ngữ nhân quả: treatment, outcome, confounder

Bài toán được phát biểu theo **suy luận nhân quả (causal inference)**. Bốn khái niệm gốc:

| Ký hiệu | Tên | Trong hệ gợi ý là gì |
|---|---|---|
| `W` | **Treatment** (biến can thiệp) | **Embedding của cặp (user, item)** `W = {(w_u, w_i)}`. Đây là "cái ta tác động/điều khiển". |
| `Y` | **Outcome** (kết quả) | **Phản hồi của user** (click/không click). |
| `U_c` | **Latent confounder** (nhiễu ẩn) | Yếu tố ẩn (độ phổ biến, đám đông...) tác động lên **cả** `W` lẫn `Y`. |
| `X` | **Pretreatment variables** | **Dữ liệu tương tác lịch sử** (chỉ positive samples) — cái có TRƯỚC treatment. |

**Dữ liệu `D` gồm 2 loại cặp** (Mục 3.1 của paper):
- **Positive pair `(u, p)`**: user `u` và item `p` mà user **đã chọn**.
- **Negative pair `(u, n)`**: user `u` và item `n` mà user **chưa tương tác** (mẫu âm).
- **`X` chỉ lấy positive samples** (vì ta cần trích CIV từ các tương tác thực đã xảy ra). Còn mẫu âm `n` dùng ở bước huấn luyện BPR (Bước 4).

**Confounder là gốc rễ của vấn đề.** `U_c` tạo ra một **"đường cửa sau" (backdoor path)** `W ← U_c → Y`: khi ta thấy `W` và `Y` cùng biến thiên, một phần là do **quan hệ nhân quả thật** `W → Y`, một phần là do **`U_c` kéo cả hai** → ta không tách được. Mô hình thường (MF) học nhầm cả phần nhiễu này thành "sở thích".

### Ba đồ thị nhân quả (Figure 1 trong paper)
- **(a) Mô hình truyền thống:** `U → Y ← I`. Coi mọi tương tác là tín hiệu thuần của sở thích → **bỏ qua nhiễu**.
- **(b) Can thiệp nhân quả:** thêm `C_i` (nhiễu tác động lên item) → mô hình hóa nhiễu, nhưng **phụ thuộc giả định đồ thị khắt khe** (phải biết trước cấu trúc nhiễu).
- **(c) Biến công cụ chuẩn:** `Z → W → Y`, `C → W`, `C → Y`. Dùng IV `Z` để gỡ nhiễu — nhưng tìm `Z` hợp lệ rất khó (xem mục 2).

---

## 2. Biến công cụ (Instrumental Variable — IV) và 3 luật nhân quả

**Ý tưởng IV:** Nếu không thể đo `U_c`, hãy tìm một biến `Z` (gọi là *công cụ*) tác động lên `W` nhưng **"sạch" khỏi nhiễu**. Khi đó, phần biến thiên của `W` *do `Z` gây ra* là phần **không dính `U_c`** → dùng phần đó để ước lượng `W → Y` không thiên lệch.

### Ba điều kiện để `Z` là IV hợp lệ (Pearl, 2009) — **đây là 3 luật cốt lõi**
1. **(Relevance — Liên quan):** `Z` **có tương quan** với treatment `W`. → `Z` phải "lay chuyển" được `W`, nếu không thì vô dụng.
2. **(Exclusion restriction — Loại trừ):** `Z` chỉ tác động lên `Y` **thông qua** `W`, **không có đường tắt** `Z → Y` trực tiếp. → mọi ảnh hưởng của `Z` lên kết quả phải "đi qua" treatment.
3. **(Unconfoundedness — Không chia sẻ nhiễu):** `Z` và `Y` **không có nhiễu chung**. → `Z` không bị `U_c` (hay biến ẩn nào) tác động cùng với `Y`.

### Vì sao IV chuẩn rất khó dùng
- Điều kiện **(1)** kiểm tra được từ dữ liệu (đo tương quan).
- Điều kiện **(2) và (3)** **KHÔNG kiểm chứng được chỉ từ dữ liệu quan sát** (Brito & Pearl, 2012). Muốn chắc, phải có **kiến thức miền** hoặc nguồn ngoại sinh (vd IV4Rec dùng *dữ liệu search* của user làm IV).
- → Tìm một IV hợp lệ là **nút thắt cổ chai**. Dữ liệu search không phải lúc nào cũng có.

---

## 3. Conditional IV (CIV) — nới lỏng luật IV

**CIV (Conditional IV):** thay vì đòi `Z` là IV "vô điều kiện", ta cho phép `Z` là IV **với điều kiện trên một tập biến `Z_c`** (gọi là *conditioning set* — tập điều kiện).

Nói cách khác: `Z` *bản thân* có thể **chưa** thỏa điều kiện (2)(3), NHƯNG **khi ta "điều kiện hóa" (conditioning) trên `Z_c`** — tức cố định/kiểm soát `Z_c` — thì `Z` **trở thành** IV hợp lệ. `Z_c` đóng vai trò **chặn các đường nhiễu** làm hỏng tính hợp lệ của `Z`.

> *(Định nghĩa CIV đầy đủ — Brito & Pearl — còn một điều kiện kỹ thuật: `Z_c` **không chứa hậu duệ của `W`**. Trong IDCIV-RS điều này tự thỏa vì `Z_c` được trích từ `X` — biến tiền can thiệp, có TRƯỚC `W`.)*

→ CIV **dễ thỏa hơn** IV chuẩn. Câu hỏi còn lại: lấy `Z` (CIV) và `Z_c` (tập điều kiện) **ở đâu ra**?

**Đóng góp chính của IDCIV-RS:** thay vì đi tìm `Z`/`Z_c` từ bên ngoài, **tự SINH (học) ra biểu diễn của CIV `Z_t` và tập điều kiện `Z_c` TRỰC TIẾP từ dữ liệu tương tác `X`** bằng VAE.

> ### ⭐ GIẢ ĐỊNH CỐT LÕI (Definition 1) — quan trọng ngang phần chứng minh
> *"Tồn tại ít nhất một CIV bên trong `X`."* `X` (lịch sử tương tác) được giả định chứa thông tin ngoại lai (vd **hành vi search** dẫn tới click) có vai trò như một công cụ. Giả định này **hợp lý** vì trong thực tế người dùng thường bị tác động bởi yếu tố ngoài (search, khuyến mãi) khiến tương tác **không khớp 100% sở thích thật**.
> **Lưu ý phản biện:** đây là **giả định KHÔNG kiểm chứng được từ dữ liệu** — toàn bộ tính hợp lệ của phương pháp dựa trên nó. (Khi trình bày nên nêu rõ đây là điểm yếu/giả định, không phải sự thật đã chứng minh.)

---

## 4. Đồ thị nhân quả của IDCIV-RS (Figure 2) — trái tim của phương pháp

Paper đề xuất DAG `G` với các biến: `X` (tiền can thiệp), `Z_t` (CIV), `Z_c` (tập điều kiện), `W` (treatment), `Y` (outcome), `U_c` (nhiễu ẩn).

**Danh sách cạnh (đọc đúng theo Mục 3.2 — đây là cách chính xác nhất để hình dung, đừng phụ thuộc hình vẽ):**

| Cạnh | Ý nghĩa |
|---|---|
| `X → Z_t`, `X → Z_c` | `Z_t`, `Z_c` đều được **trích/học từ `X`**. |
| `Z_t → W` | CIV ảnh hưởng treatment (gốc của tính *relevance*). |
| `Z_c → W` | Tập điều kiện cũng ảnh hưởng `W` (vì cũng từ `X`). |
| `Z_t → W → Y` | Ảnh hưởng của `Z_t` lên `Y` **chỉ đi qua `W`** (gốc của *exclusion*). |
| `Z_c → Y` | Tập điều kiện có đường thẳng tới `Y` (confounder ảnh hưởng kết quả). |
| `U_c → W`, `U_c → Y` | Nhiễu ẩn tác động **cả hai** → đường cửa sau `W ← U_c → Y`. |
| `U_c ⋯ Z_c` (nét đứt) | **`Z_c` có thể CHỨA/tương quan với thông tin của `U_c`** (do quan hệ phức tạp trong dữ liệu tương tác). |

> **Cạnh nét đứt `U_c ⋯ Z_c` là chìa khóa:** chính vì `Z_c` được phép "dính" `U_c`, nên khi ta **điều kiện hóa trên `Z_c`**, ta gián tiếp kiểm soát luôn ảnh hưởng của `U_c` → **chặn được đường cửa sau**. Nếu `Z_c` hoàn toàn tách rời `U_c` thì nó vô dụng trong việc gỡ nhiễu.

### 4.1. CHỨNG MINH `Z_t` là CIV hợp lệ — phần luật nhân quả quan trọng nhất

Để kiểm tra điều kiện IV, paper xét **đồ thị bị can thiệp `G_W`** (manipulated graph): **xóa mọi mũi tên ĐI RA từ `W`** (tức cắt luôn `W → Y`).

> **Vì sao phải dùng `G_W`?** Điều kiện *exclusion* (#2) đòi: ảnh hưởng của `Z_t` lên `Y` **chỉ** được phép đi qua `W`. Cách kiểm tra chuẩn là **cắt bỏ đường nhân quả `W → Y`** rồi xem **các đường CÒN LẠI** từ `Z_t` tới `Y` có bị chặn hết không. Nếu trong `G_W` mà `Z_t` đã **d-tách** khỏi `Y` (khi điều kiện trên `Z_c`), nghĩa là ngoài con đường hợp lệ qua `W` thì **không còn đường rò rỉ nào khác** — đúng tinh thần exclusion. **Mục đích của `G_W`** là **loại con đường HỢP LỆ `Z_t→W→Y` ra khỏi phép kiểm** — để bài kiểm chỉ còn xét các đường "rò rỉ" tiềm tàng (qua nhiễu, qua biến khác); nếu mọi đường rò rỉ đều bị chặn thì exclusion + unconfoundedness thỏa.

Trong `G_W`, **khi đã điều kiện trên `Z_c`**:

**(a) Tính LIÊN QUAN (điều kiện IV #1) — THỎA:**
- `Z_t` và `W` vẫn **d-connected** (nối được) khi điều kiện trên `Z_c`, **nhờ cạnh `Z_t → W`** (cạnh này không bị xóa vì nó đi VÀO `W`, không ra khỏi `W`).
- → `Z_t` thật sự lay chuyển `W`. ✓ (Relevance)

**(b) Tính LOẠI TRỪ + KHÔNG NHIỄU CHUNG (điều kiện IV #2, #3) — THỎA khi điều kiện trên `Z_c`:**

Xét tất cả các đường từ `Z_t` tới `Y` trong `G_W` (nhớ: `W → Y` đã bị cắt):

1. **Đường qua collider tại `W`:** `Z_t → W ← U_c → Y`. Trên đường này, nút `W` là **collider** (hai mũi tên đối đầu: `Z_t → W ← U_c`). **Quy tắc collider:** một đường qua collider **bị chặn mặc định**, *trừ khi* ta điều kiện trên collider đó hoặc hậu duệ của nó. Ở đây ta **không** điều kiện trên `W` (cũng không trên hậu duệ nào của `W`) → đường này **bị chặn tại collider `W`** đúng theo quy tắc collider. *(Lưu ý quan trọng: sự chặn này đúng cả trong `G` gốc lẫn `G_W` — nó do **không điều kiện trên `W`**, KHÔNG phải do thao tác cắt cạnh của `G_W`. Vai trò của `G_W` là chuyện khác — xem khung trên.)* ✓

2. **Đường fork qua `Z_c`:** `Z_t ← Z_c → Y`. Đây là cấu trúc "fork" với `Z_c` ở giữa. **Quy tắc fork:** điều kiện trên biến ở giữa fork **sẽ chặn đường**. Vì ta **đang điều kiện trên `Z_c`** → đường này **bị chặn**. ✓

→ Sau khi điều kiện trên `Z_c`, trong `G_W` **không còn đường nối mở (open path) nào** giữa `Z_t` và `Y`. Tức `Z_t` **d-separated** khỏi `Y`: ảnh hưởng của `Z_t` lên `Y` **chỉ** qua `W` (`Z_t → W → Y`), và `Z_t` không chia sẻ đường nhiễu nào với `Y`. ✓ (Exclusion + Unconfoundedness)

**Kết luận:** `Z_t` thỏa cả 3 điều kiện IV **khi điều kiện trên `Z_c`** ⇒ **`Z_t` là một CIV hợp lệ, với `Z_c` là tập điều kiện tương ứng.** Đây là nền tảng lý thuyết cho phép dùng `Z_t` để gỡ nhiễu.

> **Trực giác:** `Z_c` đóng vai "tấm chắn" — nó hứng hết các đường nhiễu (đường fork qua chính nó, và liên kết với `U_c`) để `Z_t` còn lại "sạch", chỉ tác động `Y` qua `W`. Điều then chốt (Mục 3.4) là phải làm cho **`Z_t` độc lập với `Z_c` khi cho trước `X`** (`Z_t ⊥ Z_c | X`) thì sự phân vai này mới đúng.

---

## 5. Bốn bước của IDCIV-RS (Figure 3)

### Bước 1 — Mã hóa đặc trưng (Feature encoding)
Dùng **backbone** (MF hoặc LightGCN) mã hóa (user, item) thành embedding **treatment `W_{u,i}`**:
$$W_{u,i} = \{(w_u, w_i) \mid u \in U, i \in I\}$$
Lúc này `W` còn **thô** — chứa cả sở thích thật lẫn nhiễu `U_c`.

### Bước 2 — CSEM: học `Z_t` và `Z_c` bằng VAE (Mục 3.4)
**CSEM** (CIV and conditional Set Extraction Module) = một **VAE** gồm *inference network* + *generative network*, sinh `Z_t`, `Z_c` từ `X`.

- **Inference network:** hai **encoder độc lập** ước lượng hậu nghiệm (mỗi `Z` gồm 2 phần: của **user** và của **item**):
$$q(Z_t|X) = \prod_m \mathcal{N}(\hat\mu_{Z_{t_m}}, \hat\sigma^2_{Z_{t_m}}), \qquad q(Z_c|X) = \prod_m \mathcal{N}(\hat\mu_{Z_{c_m}}, \hat\sigma^2_{Z_{c_m}})$$
- **Generative network (prior + decoder):**
  - Prior cho CIV: $p(Z_t) = \prod_m \mathcal{N}(0,1)$ (Gaussian chuẩn).
  - Prior cho tập điều kiện dùng **Conditional VAE (CVAE)**: $Z_c \sim p(Z_c \mid X)$ (lấy mẫu Monte Carlo, **có điều kiện trên `X`**).
  - Decoder tái tạo `X`: $p(X|Z_t, Z_c) = \prod_m p(X_m|Z_t,Z_c)$.
- **Hàm mất mát ELBO** (cực đại hóa cận dưới của log-likelihood):
$$\mathcal{L}_{civ} = \mathbb{E}_q[\log p(X|Z_t,Z_c)] \;-\; D_{KL}[q(Z_t|X)\,\|\,p(Z_t)] \;-\; D_{KL}[q(Z_c|X)\,\|\,p(Z_c|X)]$$

> **Chi tiết NHÂN QUẢ quan trọng nhất ở bước này:** việc dùng **CVAE để điều kiện `p(Z_c)` trên `X`** là **bước then chốt**, vì nó **được thiết kế để đạt** `Z_t ⊥ Z_c | X` (độc lập có điều kiện). Paper **khẳng định** tính chất này (Mục 3.4) và coi đó là **giả định thiết kế** — *paper không chứng minh chặt chẽ rằng VAE đảm bảo được điều đó*. Nhờ sự tách bạch này mà **`Z_t` chỉ giữ thông tin CIV (nhân quả), còn `Z_c` hút thông tin nhiễu (confounding)** — đúng vai trò đã chứng minh ở Mục 4.1. *(Đây cũng là một điểm để phản biện: tính tách bạch của VAE là kỳ vọng, không có bảo chứng lý thuyết — disentanglement của VAE nói chung không định danh được.)*

### Bước 3 — Phân rã treatment bằng bình phương tối thiểu (Mục 3.5)
Dùng CIV `Z_t` để **chiếu (project)** `W` xuống không gian "sạch", lấy phần nhân quả `\hat W`:
$$\hat W_{u,i} = f_{pro}(Z_t, W_{u,i}) = Z_t \cdot \overline{W}_{u,i}$$
trong đó `\overline{W}` là nghiệm dạng đóng của bài toán bình phương tối thiểu (least squares):
$$\overline{W}_{u,i} = \arg\min_{\overline{W}} \big\| Z_t \cdot \overline{W} - W_{u,i} \big\|_2^2 = Z_t^{\dagger} \cdot W_{u,i}$$
với `Z_t^{\dagger}` là **giả nghịch đảo Moore–Penrose** của `Z_t`.

> *(Lưu ý ký hiệu: paper dùng TRÙNG ký hiệu `W_{u,i}` cho cả treatment gốc lẫn nghiệm LS — gây rối. Ở đây ta đặt `\overline{W}` cho nghiệm LS để phân biệt rõ.)*

- **Ý nghĩa hình học (diễn giải):** đây chính là **bước 1 của 2SLS** (two-stage least squares) trong IV cổ điển — lấy "phần của `W` được dự đoán bởi công cụ `Z_t`". Vì `Z_t` sạch khỏi `U_c`, nên `\hat W` = **phần nhân quả thuần** = **sở thích thật**, tách khỏi nhiễu `U_c`.

**Tái thiết treatment** — cộng lại `Z_c`:
$$W^{re}_{u,i} = \alpha\, \hat W_{u,i} + (1-\alpha)\, Z_c$$
- Vì sao phải cộng `Z_c`? Vì **`Z_c` chặn đường nhiễu giữa `Z_t` và `Y`** — để dự đoán click **chính xác trong thế giới thực (vẫn còn nhiễu)**, mô hình cần "nhận thức" được bối cảnh nhiễu. `α` là siêu tham số cân bằng giữa **sở thích thật (`\hat W`)** và **bối cảnh nhiễu (`Z_c`)**.
- *(Giả định kỹ thuật: `\hat W` và `Z_c` cùng số chiều/không gian để cộng được.)*
- *(Biến thể **IDCIV-RS-Causal**: chỉ dùng `\hat W` (tương đương α=1) — sạch hoàn toàn nhưng recall thấp hơn một chút, vì bỏ qua bối cảnh nhiễu.)*

### Bước 4 — Dự đoán click (Mục 3.6)
Tối ưu `W^{re}` bằng **BPR loss** (Bayesian Personalized Ranking — ép điểm của positive item cao hơn negative item):
$$\mathcal{L}_{click} = -\sum_{(u,p,n)\in D} \ln \sigma\big(\langle w^{re}_u, w^{re}_p\rangle - \langle w^{re}_u, w_n\rangle\big)$$
- `p` = positive item (đã chọn), `n` = **negative item** (chưa tương tác), chọn bằng **PNSM** (Popularity-based Negative Sampling with Margin, từ DICE): *lấy mẫu âm dựa trên độ phổ biến + ngưỡng margin, để tránh chọn nhầm item mà user thực ra sẽ thích.*

**Hàm mất mát tổng:**
$$\mathcal{L}_{total} = \mathcal{L}_{civ} + \mathcal{L}_{click}$$
→ Học đồng thời: (a) tách CIV/nhiễu chuẩn (ELBO), (b) dự đoán click tốt (BPR).

---

## 6. Tóm tắt mạch logic (để trình bày)

1. **Vấn đề:** nhiễu ẩn `U_c` (độ phổ biến, đám đông) tạo đường cửa sau `W ← U_c → Y` → mô hình học tương quan giả.
2. **Công cụ nhân quả:** dùng IV để gỡ nhiễu, nhưng IV chuẩn cần 3 điều kiện mà 2 cái không kiểm chứng được từ data → khó.
3. **Nới lỏng:** dùng **CIV** (IV có điều kiện trên `Z_c`) → dễ thỏa hơn.
4. **Đột phá:** **tự học `Z_t` (CIV) + `Z_c` (tập điều kiện) từ chính `X`** bằng VAE — không cần IV ngoại sinh. (Dựa trên **giả định cốt lõi**: trong `X` tồn tại CIV.)
5. **Bảo chứng lý thuyết:** chứng minh bằng **d-separation trên đồ thị can thiệp `G_W`** rằng `Z_t` là CIV hợp lệ (liên quan tới `W`; chặn mọi đường tới `Y` ngoài qua `W`, nhờ collider tại `W` + điều kiện trên `Z_c`) — với điều kiện thiết kế `Z_t ⊥ Z_c | X` (kỳ vọng từ CVAE).
6. **Thực thi 4 bước:** mã hóa `W` → CSEM tách `Z_t,Z_c` → chiếu least squares lấy `\hat W` (sở thích thật) + tái thiết `W^{re}=α\hat W+(1-α)Z_c` → BPR dự đoán click.
7. **Kết quả:** vừa **chính xác hơn** (Recall/HR/NDCG cao hơn backbone tới ~34–40%) vừa **ít thiên lệch hơn** (IOU thấp + ổn định theo Top-K).

---

## 7. Bảng đối chiếu khái niệm (để tra nhanh khi thuyết trình)

| Khái niệm nhân quả | Định nghĩa ngắn | Vai trò trong IDCIV-RS |
|---|---|---|
| Treatment `W` | Biến ta can thiệp | Embedding cặp (user, item) |
| Outcome `Y` | Kết quả | Click / phản hồi |
| Confounder `U_c` | Nhiễu ẩn tác động cả `W` và `Y` | Độ phổ biến, đám đông (nguồn thiên lệch) |
| Backdoor path | Đường `W ← U_c → Y` gây tương quan giả | Cái cần chặn |
| IV `Z` | Biến công cụ: liên quan `W`, sạch khỏi `Y` | Cơ chế gỡ nhiễu |
| 3 điều kiện IV | Relevance, Exclusion, No-shared-confounder | Luật để `Z` hợp lệ |
| CIV `Z_t` | IV hợp lệ *khi điều kiện trên `Z_c`* (và `Z_c` không là hậu duệ `W`) | Tự học từ `X` |
| Conditioning set `Z_c` | Tập biến để điều kiện hóa, chặn nhiễu | Tự học từ `X`, hút `U_c` |
| Collider | Nút có 2 mũi tên đối đầu trên một đường (`Z_t→W←U_c`) | Chặn đường nhiễu mặc định khi không điều kiện trên nó |
| Fork | Cấu trúc `A ← B → C`; điều kiện trên `B` chặn đường | `Z_t ← Z_c → Y`: điều kiện `Z_c` chặn |
| d-separation | Hai biến bị chặn (độc lập) trên đồ thị | Công cụ chứng minh `Z_t` là CIV |
| Manipulated graph `G_W` | Đồ thị sau khi xóa mũi tên RA khỏi `W` (cắt `W→Y`) | Khung để kiểm tra điều kiện exclusion |
| 2SLS / least squares | Lấy phần `W` dự đoán bởi `Z` | Bước chiếu `\hat W = Z_t^{\dagger}W` |

---

*Ví dụ cụ thể để dễ hình dung `Z_t` / `Z_c`:* `Z_t` (CIV) ~ **động cơ ngoại lai khiến user tương tác** (vd user search ra phim rồi mới click) — phần này phản ánh ý định thật. `Z_c` (tập điều kiện) ~ **bối cảnh nhiễu** (độ phổ biến, xu hướng đám đông) khiến user click dù không hẳn thích. IDCIV-RS tách hai cái này ra, giữ `Z_t` để gợi ý đúng gu, dùng `Z_c` để "hiểu" và trừ khử thiên lệch. *(Đây chỉ là **minh họa trực giác** — VAE không định danh được nên KHÔNG có bảo chứng `Z_t`/`Z_c` mang đúng ngữ nghĩa "search"/"popularity" này.)*
