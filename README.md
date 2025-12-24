# Shopping Cart Analysis

Phân tích dữ liệu bán lẻ nhằm khám phá mối quan hệ giữa các sản phẩm thường được mua cùng nhau bằng các kỹ thuật **Association Rule Mining** như **Apriori** và **FP-Growth**.  
Project triển khai pipeline đầy đủ từ xử lý dữ liệu → khai thác luật → so sánh thuật toán → trực quan hóa kết quả.

---

## Features

- Làm sạch dữ liệu & xử lý giao dịch lỗi
- Xây dựng basket matrix (transaction × product)
- Khai thác tập mục phổ biến (Frequent Itemsets)
- Sinh luật kết hợp (Association Rules)
- Hỗ trợ 2 thuật toán:
  - Apriori
  - FP-Growth
- So sánh Apriori vs FP-Growth
- Các chỉ số đánh giá:
  - Support
  - Confidence
  - Lift
- Trực quan hóa với:
  - Bar chart
  - Scatter plot
  - Network graph
  - Biểu đồ tương tác Plotly
- Tự động hóa pipeline bằng **Papermill**
- Dashboard tương tác bằng **Streamlit**

---

## Project Structure

```text
shopping_cart_advanced_analysis/
├── data/
│   ├── processed/
│   │   ├── basket_bool.parquet
│   │   ├── cleaned_uk_data.csv
│   │   ├── invoice_weights.csv
│   │   ├── parameter_sensitivity_results.csv
│   │   ├── rules_apriori_filtered.csv
│   │   ├── rules_fpgrowth_filtered.csv
│   │   ├── rules_weighted_full.csv
│   │   └── rules_weighted_sample.csv
│   └── raw/
│       └── online_retail.csv
│
├── notebooks/
│   ├── runs/
│   │   ├── apriori_modelling_run.ipynb
│   │   ├── basket_preparation_run.ipynb
│   │   ├── compare_apriori_fpgrowth_run.ipynb
│   │   ├── compare_parameter_sensitivity_run.ipynb
│   │   ├── fp_growth_modelling_run.ipynb
│   │   └── preprocessing_and_eda_run.ipynb
│   ├── analysis_results.ipynb
│   ├── apriori_modelling.ipynb
│   ├── basket_preparation.ipynb
│   ├── fp_growth_modelling.ipynb
│   ├── preprocessing_and_eda.ipynb
│   └── weighted_association_rules.ipynb
│
├── src/
│   └── apriori_library.py
│
├── visualizations/
│   ├── chart_category_heatmap.png
│   ├── chart_num_rules.png
│   ├── chart_speedup.png
│   ├── chart_time_comparison.png
│   ├── chart_top10_weighted.png
│   └── chart_weighted_support.png
│
├── .gitignore.txt
├── LICENSE.txt
├── README.md
├── report_lab2.md
├── requirements.txt
└── run_papermill.py

```

---

## Installation

```bash
git clone <your_repo_url>
cd shopping_cart_advanced_analysis
conda create -n shopping_env python=3.11
conda activate shopping_env
pip install -r requirements.txt
```

Data Preparation
Đặt file gốc tại:

```bash
data/raw/online_retail.csv
```
File output sẽ được sinh tự động vào:

```bash
data/processed/
```

Run Pipeline (Recommended)
Chạy toàn bộ phân tích chỉ với 1 lệnh:

```bash
python run_papermill.py
```
Kết quả sinh ra:

```bash
data/processed/
├── cleaned_uk_data.csv
├── basket_bool.parquet
├── rules_apriori_filtered.csv
└── rules_fpgrowth_filtered.csv

notebooks/runs/
├── preprocessing_and_eda_run.ipynb
├── basket_preparation_run.ipynb
├── apriori_modelling_run.ipynb
├── fp_growth_modelling_run.ipynb
└── compare_apriori_fpgrowth_run.ipynb
```

### Changing Parameters
Các tham số có thể chỉnh trong `run_papermill.py` hoặc trong cell `PARAMETERS` của mỗi notebook:

```python
MIN_SUPPORT=0.02
MAX_LEN=3
FILTER_MIN_CONF=0.3
FILTER_MIN_LIFT=1.2
```
Papermill cho phép chạy pipeline với cấu hình khác nhau mà không cần sửa notebook gốc.

### Visualization & Results
Các notebook modelling hiển thị các biểu đồ:

Top luật theo Lift

Top luật theo Confidence

Scatter Support – Confidence – Lift

Network graph giữa các sản phẩm

Biểu đồ Plotly tương tác

Có thể export notebook kết quả sang HTML:

```bash
jupyter nbconvert notebooks/runs/priori_modelling_run.ipynb --to html
```

### Ứng dụng thực tế
Product recommendation

Cross-selling strategy

Combo gợi ý sản phẩm

Phân tích hành vi mua hàng

Sắp xếp sản phẩm tại siêu thị

### Tech Stack

| Công nghệ | Mục đích |
|----------|----------|
| Python | Ngôn ngữ chính |
| Pandas | Xử lý dữ liệu transaction |
| MLxtend | Apriori / FP-Growth association rules |
| Papermill | Chạy pipeline notebook tự động |
| Matplotlib & Seaborn | Visualization biểu đồ tĩnh |
| Plotly | Dashboard / biểu đồ tương tác |
| Jupyter Notebook | Môi trường notebook |

### Roadmap
Streamlit dashboard

Weighted association rules

Correlation-aware rule ranking


### Author
Project được thực hiện bởi:
Trang Le
Project được fork lại bởi:
Nhóm 6 - CNTT 17-10

📄 License
MIT — sử dụng tự do cho nghiên cứu, học thuật và ứng dụng nội bộ.
