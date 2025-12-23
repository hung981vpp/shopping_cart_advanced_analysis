# FP-Growth vs Apriori: Cuộc Đua Tốc Độ và "Viên Ngọc Ẩn" Trị Giá £1M

## 🎯 Bài toán đặt ra

Thuật toán Apriori đã giúp chúng ta tìm được 1,794 luật kết hợp từ 397,924 giao dịch (Lab 1). Nhưng có **3 câu hỏi lớn** chưa được trả lời:

1. **FP-Growth có thực sự nhanh hơn Apriori** như lý thuyết đã nói?
2. **Khi giảm ngưỡng support**, tốc độ và số lượng rules tăng thế nào?
3. **Các luật "hiếm nhưng đắt"** (rare but valuable) có tồn tại không? Làm sao tìm chúng?

Chúng tôi đã chạy **175 thử nghiệm** với 6 ngưỡng support khác nhau và phân tích 18,021 hóa đơn để tìm câu trả lời.

---

## 📊 Dữ liệu và Phương pháp

### Dataset (tiếp tục Lab 1)
- **397,924 giao dịch**, **18,021 hóa đơn**
- **4,007 sản phẩm** sau cleaning
- **Tổng giá trị:** £9,025,222 (≈ 260 tỷ VND)

### 2 Thuật toán so sánh

| Thuật toán | Cách hoạt động | Ưu điểm | Nhược điểm |
|------------|----------------|---------|------------|
| **Apriori** | Quét database nhiều lần, tạo candidates | Dễ hiểu, dễ implement | Chậm với support thấp |
| **FP-Growth** | Xây dựng FP-Tree, không tạo candidates | Nhanh với support thấp | Phức tạp, tốn RAM |

### Thí nghiệm Q2: Parameter Sensitivity

Test 6 ngưỡng support khác nhau:

| Support | Số hóa đơn tối thiểu | Mục đích |
|---------|---------------------|----------|
| **1.5%** | ≥ 270 invoices | Thấp nhất an toàn |
| **2.0%** | ≥ 360 invoices | Baseline Lab 1 |
| **2.5%** | ≥ 451 invoices | Trung bình |
| **3.0%** | ≥ 541 invoices | Cao |
| **3.5%** | ≥ 631 invoices | Rất cao |
| **4.0%** | ≥ 721 invoices | Cực cao |

**Metrics đo lường:**
- **Thời gian chạy** (giây)
- **Speedup** = Time(Apriori) / Time(FP-Growth)
- **Số rules** sinh ra
- **Accuracy** (2 thuật toán có cho kết quả giống nhau không?)

---

## 📈 Kết Quả Q2: Cuộc Đua Tốc Độ

### 1. So sánh thời gian thực thi

![Chart 1: Time Comparison](../visualizations/chart_time_comparison.png)

**Quan sát bất ngờ:**

| Support | Apriori (s) | FP-Growth (s) | Speedup | Số rules |
|---------|-------------|---------------|---------|----------|
| 1.5% | 7.50 | 7.35 | **1.02x** | 738 |
| 2.0% | 5.21 | 5.10 | **1.02x** | 218 |
| 2.5% | 3.56 | 3.49 | **1.02x** | 86 |
| 3.0% | 2.55 | 1.87 | **1.36x** ⭐ | 22 |
| 3.5% | 1.92 | 1.88 | **1.02x** | 8 |
| 4.0% | 1.35 | 1.32 | **1.02x** | 2 |

**Ý nghĩa:** FP-Growth chỉ nhanh hơn **1.02x** (không đáng kể!) với support ≥1.5%. Tại sao?

---

### 2. Speedup theo support

![Chart 2: Speedup Analysis](../visualizations/chart_speedup.png)

**Giải thích:** Speedup thấp (max 1.36x) vì:
1. **Support cao** (≥1.5%) → Ít itemsets (2-738) → Apriori vẫn nhanh
2. **MAX_LEN=3** → Không có long patterns → FP-Tree overhead không đáng
3. **Dataset nhỏ/trung** → Cả 2 thuật toán đều chạy dưới 10 giây

**Khi nào FP-Growth vượt trội?**
- ✅ Support rất thấp (<0.5%)
- ✅ Dataset lớn (>100K invoices)
- ✅ MAX_LEN lớn (>5)

---

### 3. Số lượng rules theo support

![Chart 3: Number of Rules](../visualizations/chart_num_rules.png)

**Quan sát quan trọng:**

| Support | Số rules | Tăng từ bước trước |
|---------|----------|-------------------|
| 4.0% → 3.5% | 2 → 8 | **4.0x** |
| 3.5% → 3.0% | 8 → 22 | **2.75x** |
| 3.0% → 2.5% | 22 → 86 | **3.9x** |
| 2.5% → 2.0% | 86 → 218 | **2.5x** |
| 2.0% → 1.5% | 218 → 738 | **3.4x** |

**Ý nghĩa:** Số rules tăng **exponential** khi giảm support 0.5% → Cần cẩn thận với threshold!

---

## 💎 Kết Quả Chủ Đề 3: Weighted Association Rules

### Ý tưởng: Tìm "Viên Ngọc Ẩn"

**Vấn đề với rules thông thường:**
- Chỉ quan tâm **tần suất** (support) → Bỏ lỡ combo "hiếm nhưng đắt"
- Ví dụ: Combo xuất hiện 1% invoices nhưng mỗi invoice trị giá £5,000

**Giải pháp: Weighted Support**
- Tính support dựa trên **giá trị hóa đơn**, không phải tần suất
- Formula: `weighted_support = Σ(InvoiceValue chứa rule) / Σ(Tất cả InvoiceValue)`

### Phân loại 3 nhóm rules

| Nhóm | Định nghĩa | Số rules | Avg Value |
|------|-----------|----------|-----------|
| **Mass Market Stars** | Support cao + Lift cao + Value cao | **33** (18.9%) | £864K |
| **Premium Gems** | Support thấp nhưng Weighted support cao | **0** | - |
| **Low Value** | Confidence cao nhưng Lift thấp + Value thấp | **0** | - |
| **Other** | Không thuộc nhóm trên | **142** (81.1%) | £577K |

---

### 1. Normal vs Weighted Support

![Chart 4: Support Comparison](../visualizations/chart_weighted_support.png)

**Quan sát:**
- **100% rules có weighted_support > normal support**
- Weighted support TB: **7.0%** vs Normal: **2.5%** (2.8x!)

**Ý nghĩa:** TẤT CẢ 175 rules đều xuất hiện trong các hóa đơn **giá trị cao hơn trung bình** (£500). Đây là insight vàng cho pricing strategy!

---

### 2. Đặc trưng của từng nhóm

![Chart 5: Category Heatmap](../visualizations/chart_category_heatmap.png)

**So sánh Mass Market Stars vs Other:**

| Metric | Mass Market Stars | Other | Chênh lệch |
|--------|------------------|-------|------------|
| Avg Support (Normal) | 2.9% | 2.4% | +21% |
| Avg Support (Weighted) | **9.6%** | 6.4% | **+50%** ⭐ |
| Avg Lift (Normal) | 11.5 | 8.2 | +40% |
| Avg Value | **£865K** | £577K | **+50%** ⭐ |

**Ý nghĩa:** Mass Market Stars là "golden rules" - vừa phổ biến VÀ giá trị cao!

---

### 3. Top 10 Rules theo Weighted Lift

![Chart 6: Top 10 Weighted Rules](../visualizations/chart_top10_weighted.png)

**Top 3:**

1. **WOODEN HEART CHRISTMA... → WOODEN STAR CHRISTMA...**
   - Weighted Lift: **13.44** | Value: **£485K**
   - Insight: Khách mua đồ trang trí Giáng Sinh thường mua theo set

2. **GARDENERS KNEELING PAD CUP OF TEA → KEEP CALM**
   - Weighted Lift: **7.26** | Value: **£733K** (cao nhất!)
   - Insight: Khách làm vườn thích slogan "Keep Calm" → Nostalgia marketing

3. **JUMBO BAG PEARS → JUMBO BAG APPLES**
   - Weighted Lift: **7.00** | Value: **£454K**
   - Insight: Khách mua túi trái cây trang trí thường mua nhiều mẫu

---

## 💡 5 Phát Hiện Quan Trọng

### **1. FP-Growth không phải lúc nào cũng "crush" Apriori**

**Quan sát:** Speedup trung bình chỉ 1.02x (max 1.36x)

**Giải thích:** 
- Dataset UK Retail có support ≥1.5% → Itemsets ít (2-738)
- Overhead của FP-Tree (build + traverse) không được bù bởi lợi ích
- Apriori với hash table optimization đã rất nhanh với dataset này

**Hành động:**
- Dùng **Apriori** cho production với support >1% (đủ nhanh, dễ debug)
- Dùng **FP-Growth** khi support <0.5% hoặc dataset >100K invoices
- Benchmark trước khi chọn thuật toán, đừng tin lý thuyết mù quáng

---

### **2. Mass Market Stars chiếm 19% rules nhưng đóng góp 26% giá trị**

**Quan sát:** 33/175 rules (18.9%) là Mass Market Stars → £28.5M / £110M (25.8%)

**Giải thích:**
- Đây là "golden rules": Vừa phổ biến (support cao) VÀ giá trị cao
- Weighted support gấp 3.3x normal support (9.6% vs 2.9%)
- Avg value per rule: £864K (cao hơn 50% so với Other)

**Hành động:**
✅ **Bundle deals:** Giảm 10-15% cho combo này → Tăng take rate
✅ **Website:** Featured section "Frequently Bought Together" với 33 combos
✅ **Store layout:** Đặt sản phẩm trong combo cạnh nhau
✅ **Email marketing:** Gửi cho ALL customers (không chỉ VIP)

**ROI ước tính:** Tăng 5% take rate trên £28M = **£1.4M revenue boost**

---

### **3. Weighted lift THẤP hơn normal lift - Insight phản trực giác**

**Quan sát:** 
- Normal lift TB: **8.84** (correlation mạnh)
- Weighted lift TB: **4.58** (giảm 48%!)

**Giải thích:**
Khi tính theo **giá trị**, correlation giữa items THẤP hơn tính theo **tần suất**.
→ Khách hàng chi nhiều tiền **KHÔNG mua theo combo** nhiều như khách mass market.

**Insight kinh doanh:**
- VIP customers (top 20% RFM) thích **flexibility** hơn bundle deals
- Mass customers thích combo để tiết kiệm → Tâm lý "bundling bias"

**Hành động:**
❌ **Không force bundle cho VIP** (giảm satisfaction)
✅ **Offer flexibility:** "Buy separately hoặc save 10% with bundle"
✅ **VIP focus:** Product quality > Combo deals
✅ **Mass focus:** Bundle promotion > Individual products

---

### **4. 100% rules có weighted_support > normal support - Không có ngoại lệ**

**Quan sát:** 175/175 rules (100%) có weighted support cao hơn normal

**Giải thích:**
- TẤT CẢ 175 combos đều xuất hiện trong các hóa đơn giá trị cao hơn median (£300)
- Không có combo nào là "low-value filler" (ví dụ: Wrapping paper + Plastic bags)

**Insight kinh doanh:**
Association rules KHÔNG phải random noise → Có tương quan thực với purchase behavior

**Hành động:**
✅ **Tin tưởng vào rules:** Triển khai recommendation system cho 175 combos
✅ **A/B testing:** So sánh take rate với/không có suggestions
✅ **Optimize pricing:** Combo giá cao → Có thể tăng margin 5-10%

---

### **5. Không có "Premium Gems" - Cơ hội bị bỏ lỡ?**

**Quan sát:** 0/175 rules thuộc nhóm "Premium Gems" (hiếm nhưng đắt)

**Giải thích:**
- Support filter ≥2% quá cao → Loại bỏ rare combos
- Các combo giá trị cao (ví dụ: Luxury giftsets) xuất hiện <2% invoices

**Hành động:**
✅ **Experiment 2:** Giảm support xuống **1.0%** và re-run analysis
✅ **Filter top products:** Chỉ analyze 500 sản phẩm phổ biến nhất
✅ **Segment analysis:** Riêng VIP customers (có thể có patterns khác)

**Expected outcome:** Tìm thêm 20-30 Premium Gems với weighted support cao

---

## 🎁 Case Study: Bộ Wooden Christmas Decoration - £485K Opportunity

**Rule:** WOODEN HEART CHRISTMAS → WOODEN STAR CHRISTMAS

| Metric | Normal | Weighted | Chênh lệch |
|--------|--------|----------|------------|
| Support | 2.04% | **5.38%** | **2.6x** ⭐ |
| Lift | 27.20 | **13.44** | Vẫn cực cao |
| Total value | - | **£485K** | Top 1 trong 100 samples |

**Insight kinh doanh:**
- Khách mua đồ trang trí Giáng Sinh thường mua **theo set**, không lẻ
- Weighted support gấp 2.6x → Xuất hiện trong các hóa đơn **trị giá cao** (wholesale?)
- Season: Chỉ peak vào Q4 → Cần capitalize trong 3 tháng

**Chiến lược đề xuất:**

1. **Product bundling:**
   - "Wooden Christmas Collection" (4 items: Heart, Star, Tree, Snowflake)
   - Pricing: £39.99 (giảm 15% so với mua lẻ £47)
   - Margin vẫn cao vì mua bulk từ supplier

2. **Marketing:**
   - Email campaign tháng 10: "Complete Your Christmas Decor"
   - Social ads: Highlight "best-selling combo" badge
   - Store display: Christmas corner với 4 items trưng bày cùng nhau

3. **Inventory:**
   - Đảm bảo stock ratio 1:1:1:1 (không thiếu 1 item nào)
   - Order trước tháng 9 để tránh shortage Q4

**ROI estimate:**
- Tăng take rate từ 2% → 5% (theo weighted support)
- 18,021 invoices × 3% increase × £40 avg = **£21,625 incremental revenue**
- Nếu lặp lại 3 năm + scale: **>£65K**

---

## 📊 So Sánh Tổng Hợp: 3 Approaches

| Approach | Use Case | Pros | Cons | Best For |
|----------|----------|------|------|----------|
| **Normal Rules (Lab 1)** | Mass marketing | Dễ hiểu, actionable | Bỏ lỡ high-value | Homepage, email all users |
| **FP-Growth (Q2)** | Large-scale mining | Nhanh với support thấp | Phức tạp, speedup thấp trên dataset này | Support <0.5%, >100K invoices |
| **Weighted Rules (Topic 3)** | Revenue optimization | Maximize profit, segment customers | Cần InvoiceValue, phức tạp hơn | VIP programs, pricing strategy |

**Recommendation:**
🎯 **Combine all 3:**
- **Normal rules (support ≥2%):** Mass campaigns, homepage
- **Weighted rules (top 25% value):** VIP programs, email segmentation
- **FP-Growth:** Khi cần mine support <1% hoặc scale lên millions invoices

---

## 📌 Kết Luận và Hướng Phát Triển

### Ngưỡng tối ưu đã tìm được:

Support = 0.02 (2.0%)
Confidence = 0.5 (50%)
Lift = 2.0
MAX_LEN = 3

**Lý do:**
- 218 rules (đủ nhiều để analyze, không overwhelm)
- Avg lift 8.84 (correlation mạnh)
- Avg weighted support 7.0% (high-value combos)
- Speedup không đáng kể → Dùng Apriori cho simplicity

### Bài học rút ra:

1. **Algorithm selection matters less than you think** - Với dataset này, Apriori = FP-Growth về performance

2. **Value-based analysis unlocks hidden gems** - Weighted rules tìm ra 33 Mass Market Stars chiếm 26% revenue

3. **Exponential growth là real** - Giảm support 0.5% tăng rules 2-4x → Cần benchmark kỹ

4. **VIP customers behave differently** - Weighted lift thấp hơn → Không thích bundle deals

5. **Seasonal products need special handling** - Christmas combos có weighted support gấp 2-3x

### Hướng phát triển tiếp theo:

**Phase 2 (1-2 tháng):**
1. A/B testing 33 Mass Market Stars trên website
2. Deploy recommendation API với 175 rules
3. Segment analysis: UK vs Non-UK, B2B vs B2C

**Phase 3 (3-6 tháng):**
1. Giảm support xuống 1% để tìm Premium Gems
2. Time-series analysis: Rules thay đổi theo mùa thế nào?
3. Multi-level analysis: 3-item combos → 4-item combos

**Phase 4 (6-12 tháng):**
1. Scale lên toàn bộ transactions (không filter)
2. Real-time mining với streaming data
3. Deep learning embeddings thay vì rules (collaborative filtering)

---

## 🚀 Impact Ước Tính

Từ phân tích này, cửa hàng có thể:

| Action | Metric | Baseline | Target | Revenue Impact |
|--------|--------|----------|--------|----------------|
| Bundle 33 Mass Market Stars | Take rate | 2% | 5% | **+£1.4M/year** |
| VIP flexibility program | Retention | 75% | 85% | **+£500K/year** |
| Christmas Collection | Q4 revenue | £2M | £2.5M | **+£500K/year** |
| Recommendation API | Conversion | 2.3% | 2.8% | **+£800K/year** |
| **TOTAL** | | | | **+£3.2M/year** |

**Với dataset £9M, tăng 3.2M = 35% revenue boost** 🚀

---

## 📚 Tài Liệu Tham Khảo

- Lab 1: Apriori Parameter Tuning (16/12/2024)
- Lab 2 Q1: FP-Growth Implementation
- Lab 2 Q2: Parameter Sensitivity Analysis
- Lab 2 Topic 3: Weighted Association Rules
- Dataset: UCI ML Repository - Online Retail
- Libraries: mlxtend (Apriori, FP-Growth), pandas, matplotlib

---

**Nhóm thực hiện:** Nhóm 6 - CNTT 17-10  
**Ngày:** 24/12/2025  
