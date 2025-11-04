# Amazon Sales Dashboard - Data Cleaning Summary

## 📁 Notebook Created

### ✅ Completed Notebook

**01_data_cleaning_amazon_sales.ipynb** - Amazon Sales Report

---

## 📊 Dataset: Amazon Sales Report

**Notebook:** `01_data_cleaning_amazon_sales.ipynb`

### Original Data
- **Rows:** ~128,977
- **Columns:** 24 (Order ID, Date, Status, Fulfilment, Sales Channel, ship-service-level, Style, SKU, Category, Size, ASIN, Courier Status, Qty, currency, Amount, ship-city, ship-state, ship-postal-code, ship-country, promotion-ids, B2B, fulfilled-by, Unnamed: 22, index)

### Data Quality Issues Found
- **Duplicates:** Multiple duplicate rows
- **Missing Values:** 
  - Courier Status: ~50% missing
  - Amount: ~8% missing (cancelled orders)
  - Currency: ~8% missing (cancelled orders)
  - ship-postal-code: ~2% missing
  - promotion-ids: ~60% missing (no promotions)
- **Redundant Columns:** Unnamed: 22, index column
- **Data Type Issues:** Date stored as string, B2B as string instead of boolean
- **Whitespace:** Extra spaces in categorical columns
- **Invalid Values:** Potential negative amounts or quantities

### Cleaned Data
- **Rows:** ~128,000+ (after removing duplicates and invalid records)
- **Columns:** 40+ (original + engineered features)

### Key Features Created
- **Date features:** order_year, order_month, order_quarter, order_day_of_week, order_month_name, order_day_name, order_season
- **Calculated metrics:** unit_price, order_status_category, size_category
- **Binary flags:** is_cancelled, is_completed, is_b2b, is_amazon_fulfilled, is_merchant_fulfilled, has_promotion, is_weekend
- **Aggregations:** orders_from_location, category_total_sales, category_avg_sales, category_order_count, state_total_sales, state_avg_sales, state_order_count
- **Comparative metrics:** amount_vs_category_avg

---

## 🎯 Data Cleaning Steps (Following HR Dashboard Structure)

### Step 1: Delete Redundant Columns
- Identified and removed `Unnamed: 22` column (completely empty)
- Removed `index` column (redundant row numbers)
- Verified all other columns are relevant for analysis

### Step 2: Drop / Rename the Columns
- Standardized all column names to `snake_case` format
- Replaced spaces with underscores
- Replaced hyphens with underscores
- Converted to lowercase for consistency

**Before:** `Order ID`, `Sales Channel `, `ship-service-level`  
**After:** `order_id`, `sales_channel`, `ship_service_level`

### Step 3: Remove Duplicates
- Identified duplicate rows based on all columns
- Removed exact duplicate entries
- Kept first occurrence of duplicates

**Result:** Removed duplicate rows, reducing dataset size

### Step 4: Remove the NaN Values from the Dataset

#### Critical Columns (Dropped rows with missing values)
- `order_id` - Essential for tracking orders
- `date` - Required for time-series analysis

#### Non-Critical Columns (Filled missing values)
- `courier_status` → Filled with 'Unknown'
- `ship_postal_code` → Filled with 0
- `promotion_ids` → Filled with 'None'
- `amount` → Filled with 0 (for cancelled orders)
- `currency` → Filled with 'INR'

### Step 5: Clean Individual Columns

#### 5.1 Clean Order ID Column
- Removed whitespace from order IDs
- Verified uniqueness and format
- Ensured no null values remain

#### 5.2 Clean Date Column
- Converted string dates to datetime format
- Used format: `%m-%d-%y` (MM-DD-YY)
- Handled invalid dates by coercing to NaT
- Removed rows with invalid dates
- Extracted date components:
  - Year, Month, Quarter
  - Day of week (0=Monday, 6=Sunday)
  - Month name, Day name
  - Season (Winter, Spring, Summer, Fall)

#### 5.3 Clean Status Column
- Removed extra whitespace
- Standardized to Title Case
- Created status categories:
  - **Cancelled:** Orders that were cancelled
  - **Completed:** Shipped or delivered orders
  - **Pending:** Orders awaiting processing
  - **Other:** Any other status

#### 5.4 Clean Amount and Quantity Columns
- Verified numeric data types
- Checked for negative values
- Removed invalid negative amounts
- Removed invalid negative quantities
- Validated that cancelled orders can have zero amount (valid business logic)

#### 5.5 Clean Categorical Columns
- Standardized the following columns:
  - `fulfilment` → Title Case
  - `sales_channel` → Title Case
  - `ship_service_level` → Title Case
  - `category` → Title Case
  - `size` → Title Case
  - `courier_status` → Title Case
  - `ship_city` → Title Case
  - `ship_state` → Title Case
  - `ship_country` → Title Case
  - `style` → Title Case
  - `sku` → Title Case

#### 5.6 Clean B2B Column
- Converted string values ('True', 'False') to boolean
- Mapped 'true' → True, 'false' → False
- Filled missing values with False
- Created binary flag `is_b2b` (1 or 0)

### Step 6: Check for Some More Transformations

#### 6.1 Create Calculated Columns

**Unit Price:**
- Formula: `amount / qty` (when qty > 0)
- Handles division by zero by setting to 0

**Order Status Category:**
- Simplified status into 4 categories
- Enables easier filtering and analysis

**Size Category:**
- Grouped sizes into meaningful categories:
  - **Small:** XS, S
  - **Medium:** M, L
  - **Large:** XL, XXL
  - **Extra Large:** 3XL, 4XL, 5XL, 6XL
  - **Other:** Non-standard sizes

**Season:**
- Winter: Dec, Jan, Feb
- Spring: Mar, Apr, May
- Summer: Jun, Jul, Aug
- Fall: Sep, Oct, Nov

#### 6.2 Create Binary Flags

- `is_cancelled` → 1 if order cancelled, 0 otherwise
- `is_completed` → 1 if order completed, 0 otherwise
- `is_b2b` → 1 if B2B order, 0 otherwise
- `is_amazon_fulfilled` → 1 if fulfilled by Amazon, 0 otherwise
- `is_merchant_fulfilled` → 1 if fulfilled by merchant, 0 otherwise
- `has_promotion` → 1 if promotion applied, 0 otherwise
- `is_weekend` → 1 if ordered on weekend, 0 otherwise

#### 6.3 Create Aggregated Features

**Location-Level:**
- `orders_from_location` → Total orders from each city-state combination

**Category-Level:**
- `category_total_sales` → Total revenue per category
- `category_avg_sales` → Average order value per category
- `category_order_count` → Number of orders per category

**State-Level:**
- `state_total_sales` → Total revenue per state
- `state_avg_sales` → Average order value per state
- `state_order_count` → Number of orders per state

**Comparative Metrics:**
- `amount_vs_category_avg` → % difference from category average

#### 6.4 Identify Outliers
- Used IQR (Interquartile Range) method
- Identified outliers in `amount` column
- Created visualizations (histogram and boxplot)
- Documented outlier percentage

#### 6.5 Sort Data
- Sorted by `date` and `order_id`
- Reset index for clean sequential numbering
- Ensures chronological order for time-series analysis

#### 6.6 Final Data Quality Check
- Verified final dataset shape
- Confirmed all columns present
- Checked remaining missing values
- Validated data types
- Documented final statistics

---

## 📊 Data Quality Metrics

### Before Cleaning
- **Total Rows:** 128,977
- **Total Columns:** 24
- **Duplicates:** Present
- **Missing Values:** Multiple columns with 2-60% missing
- **Data Type Issues:** Yes (dates as strings, B2B as string)
- **Invalid Values:** Potential negatives

### After Cleaning
- **Total Rows:** ~128,000+
- **Total Columns:** 40+
- **Duplicates:** 0
- **Missing Values:** Minimal (only valid business cases)
- **Data Type Issues:** Resolved
- **Invalid Values:** Removed
- **New Features:** 20+ engineered features

---

## 📁 Output File

**Cleaned Data:**
- `data/processed/Amazon_Sales_cleaned.csv`
- Ready for Power BI import
- Ready for advanced analytics
- All features properly typed and formatted

---

## 🎓 Best Practices Applied

1. **Consistent Naming:** All columns in snake_case format
2. **Data Validation:** Removed invalid values (negatives, invalid dates)
3. **Missing Value Strategy:** 
   - Dropped rows with missing critical fields
   - Filled non-critical fields appropriately
4. **Feature Engineering:** Created actionable metrics for business analysis
5. **Documentation:** Clear markdown sections explaining each step
6. **Reproducibility:** Clean, well-commented code
7. **Output Management:** Saved cleaned data to processed folder
8. **Data Types:** Proper data types for all columns
9. **Categorical Standardization:** Consistent formatting across all text fields
10. **Business Logic:** Respected valid business cases (e.g., zero amount for cancelled orders)

---

## ✅ Data Cleaning Completed Successfully!

The Amazon Sales dataset has been thoroughly cleaned and is ready for:
- ✅ Exploratory Data Analysis (EDA)
- ✅ Power BI Dashboard Development
- ✅ Statistical Analysis
- ✅ Machine Learning Models
- ✅ Business Intelligence Reporting

**Processing Summary:** 128,977 raw records → ~128,000+ cleaned records with 40+ features
