# Solar Radiation Data Processing and Forecasting

## Overview
โครงการนี้นำข้อมูลจากแหล่งต่างๆ เช่น ERA5 และ CAMS มาทำการวิเคราะห์และเปรียบเทียบ เพื่อศึกษาปัจจัยที่ส่งผลต่อค่าความเข้มแสงอาทิตย์ (Global Horizontal Irradiance - I) ทั้งในแง่ของการแสดงผลเชิงเวลา และการวิเคราะห์เชิงสถิติ

โดยมุ่งเน้นไปที่:
- การเปรียบเทียบข้อมูลรังสีจากแต่ละแหล่งในช่วงเวลาเดียวกัน
- การประเมินความแม่นยำของแต่ละแหล่งข้อมูลด้วยตัวชี้วัด เช่น MAE, RMSE, MBE และ Correlation
- การนำเสนอข้อมูลด้วยกราฟเปรียบเทียบเพื่อให้เข้าใจแนวโน้มและความแตกต่าง

## Data Preparing
ข้อมูลดิบที่ได้จาก dataset ต่างๆ จะเก็บอยู่ในโฟลเดอร์  `data` โดยจะมีอยู่หลักๆ 3 ส่วนได้แก่

- **กรมพัฒนาพลังงานทดแทนและอนุรักษ์พลังงาน (DeDe)** :
  - ตัวแปรที่ใช้ : ค่ารังสี
  - รูปแบบ: csv
  - ข้อมูลจากปี 2023
  - มี timestamp แบบ UTC+7
  - ความละเอียดของข้อมูล 15 นาที

- **ECMWF Reanalysis v5 (ERA5)** :
  - ตัวแปรที่ใช้ : ssrd (J/m<sup>2</sup>) 
  - รูปแบบ: NetCDF
  - ข้อมูลจากปี 2023
  - มี timestamp แบบ UTC+0
  - ความละเอียดของข้อมูล 1 ชั่งโมง

- **Copernicus Atmosphere Monitoring Service (CAMS)** :
  
  ประกอบไปด้วย 2 ส่วนก็คือ

   **1. solar radiation time-series**
  - ตัวแปรที่ใช้ : GHI (Wh/m<sup>2</sup>)
  - รูปแบบ: CSV
  - ข้อมูลจากปี 2023
  - มี timestamp แบบ UTC+0
  - ความละเอียดของข้อมูล 1 ชั่งโมง และ 15 นาที
  
   **2. global atmospheric composition forecasts**
  - ตัวแปรที่ใช้ : ssrd (J/m<sup>2</sup>)
  - รูปแบบ: grib
  - ข้อมูลจากปี 2023
  - มี timestamp แบบ UTC+0
  - ความละเอียดของข้อมูล 1 ชั่งโมง

## Data Processing
ขั้นตอนการประมวลผลข้อมูลแบ่งออกเป็นสองส่วนหลัก คือ การประมวลผลข้อมูลจาก ERA5 และจาก CAMS เพื่อให้พร้อมสำหรับการเปรียบเทียบและวิเคราะห์ร่วมกัน

---

####  1. ERA5 Processing

- ดึงข้อมูลตามตำแหน่งสถานี (`lat`, `lon`) โดยใช้ `method='nearest'`
- แปลง `UTC` → `Asia/Bangkok`
- คัดเฉพาะข้อมูลของปี 2023
- แปลงเป็นรายชั่วโมง (`1H`) และราย 15 นาที (`15min`) ด้วย `resample()`
- เติมค่าขาดหาย (`NaN`) ด้วย `interpolate(method='linear')`
- บันทึกเป็น `.csv` สำหรับแต่ละสถานี

---

####  2. CAMS Processing

- แปลงข้อมูลจาก JSON หรือ CSV ให้อยู่ในรูปแบบ DataFrame
- แปลง `UTC` → `Asia/Bangkok`
- กรองเฉพาะปี 2023
- หากจำเป็น: แปลงจากรายชั่วโมง → 15 นาที ด้วย `resample()` และ `interpolate()`
- จัดระเบียบคอลัมน์ เช่น `Datetime`, `GHI`
- บันทึกเป็น `.csv` แยกตาม site เช่นเดียวกับ ERA5

---

#### 3. CAMS (Forecast) Processing

- แปลงข้อมูลจาก JSON หรือ CSV ให้อยู่ในรูปแบบ DataFrame
- แปลง `UTC` → `Asia/Bangkok`
- กรองเฉพาะปี 2023
- หากจำเป็น: แปลงจากรายชั่วโมง → 15 นาที ด้วย `resample()` และ `interpolate()`
- จัดระเบียบคอลัมน์ เช่น `Datetime`, `GHI`
- บันทึกเป็น `.csv` แยกตาม site เช่นเดียวกับ ERA5

---

### Output
- ข้อมูล ERA5 และ CAMS จะถูกจัดเก็บไว้ในรูปแบบที่เหมือนกัน
- พร้อมสำหรับการนำไปเปรียบเทียบเชิงสถิติและการวิเคราะห์กราฟ
## Preprocessed Data
- **ERA5** 
  - clound 
  - radiation
    - era5_time_series_15min
    - era5_time_series_1hr
- **CAMS** 
  - cams_time_series_15min
  - cams_time_series_1hr
  - cams_forecast
    - cams_forecast_15min
    - cams_forecast_1hr
## Merged Processed Data
รวมข้อมูลจากไซต์ต่างๆ เข้าด้วยกันเเล้วตัด column ที่ไม่จำเป็นทิ้ง
- merge ด้วย datetime
- ลบ column ที่ไม่เกี่ยวกับ I (ความเข้มเเสงอาทิตย์) 
- คำนวณ Clear Sky Index (สำหรับ cloud)

### Output
- ERA5(ssrd) CAMS(GHI) DeDe(I,Short-wave irradiation) เเบบ 15 min เเละ 1 hr
- CAMS_Forecast(I) DeDe(I) เเบบ 15 min เเละ 1 hr
- ERA5(cloud) DeDe(cloud) Clear Sky Index เเบบ 1 hr
## Visualization and Analysis
หลังจากรวมข้อมูลและคำนวณตัวชี้วัดเสร็จเรียบร้อย ได้มีการสร้างกราฟและวิเคราะห์เปรียบเทียบผลของ CAMS และ ERA5 สำหรับแต่ละสถานี โดยแบ่งออกเป็นหมวดหมู่ดังนี้:

### 1️. Time Series Plot

- เปรียบเทียบค่าความเข้มแสงอาทิตย์ (GHI) ของ CAMS ERA5 DeDe Soda ราย 15 นาที
- วิเคราะห์ค่ารังสีแตกต่างกัน
  
![plot](./figure/ts_1h_1.png)

### 2️. Scatter Plot

- ERA5(cloud) & DeDe(CI_RGB) vs DeDe(I)
- ใช้ดูความสัมพันธ์เเละเเน้วโน้มของสองตัวเเปร
  
![plot](./figure/c_1.png)
### 3️. Matrix Score

- กรองข้อมูลที่ไม่จำเป็น เเละที่ค่า I > 10 (W/m<sup>2</sup>)
- เปรียบเทียบค่าทางสถิติของ CAMS เเละ ERA5 โดยใช้ DeDe เป็นค่าจริง
- ค่าทางสถิติมี MAE,RMSE,MBE,nMAE,nRMSE,nMBE,correlation
- Mean Absolute Error (MAE) : ค่าความคลาดเคลื่อนเฉลี่ย วัดค่าความผิดพลาดโดยไม่สนใจทิศทาง ยิ่งค่า MAE ต่ำ แสดงว่าโมเดลแม่นยำ
- Normalized Mean Absolute Error (nMAE) : MAE ที่แปลงเป็นร้อยละเพื่อให้สะดวกต่อการอ่านข้อมูล 
- Root Mean Square Error (RMSE) : ค่ารากที่สองของความคลาดเคลื่อนเฉลี่ยกำลังสอง วัดความผิดพลาดแบบให้น้ำหนักกับค่าคลาดเคลื่อนมากๆ เหมาะสำหรับดูว่ามีจุด outlier เยอะไหม
- Normalized Root Mean Square Error (nRMSE) : RMSE ที่แปลงเป็นร้อยละเพื่อให้สะดวกต่อการอ่านข้อมูล 
- Mean Bias Error (MBE) : ค่าความเอนเอียงของโมเดล ค่าบวก โมเดล ประเมินค่าสูงเกินจริง ค่าลบ  โมเดล ประเมินค่าต่ำเกินไป
- Normalized Mean Bias Error (nMBE) : ค่าความเอนเอียงในรูปแบบร้อยละเพื่อให้สะดวกต่อการอ่านข้อมูล 
- Correlation ดูความสอดคล้องของสองตัวเเปร มีค่าตั้งเเต่ -1 ถึง 1

## Conclusion
- ได้ข้อมูลที่ Clean ของกลุ่มตัวเเปร Cloud Radiation เเละ Forecast
- กราฟ Time Series Radiation, Time Series Forecast Radiation, Matric Score, Scatter Plot Cloud 

## Forecast today
- ดึงข้อมูลการพยากรณ์รังสีแสงอาทิตย์ (SSRD) จาก CAMS นำมาประมวลผล เเล้วแสดงผลในรูปแบบกราฟ Time Series โดยมีการปรับเวลาตามเวลาไทย (UTC+7) 

# Structure
## Code 
เก็บ Code ทั้งหมดของ Project มีดังนี้
 - CAMS_download.ipynb
 - CAMS_proceed_data.ipynb
 - ERA5_proceed_data.ipynb
 - cloud_visualize.ipynb
 - forecast_proceed_data.ipynb
 - forecast_today.ipynb
 - matrix_score.ipynb
 - merge_data.ipynb
 - solar_rad_visualize.ipynb
## Data
เก็บข้อมูลดิบของ Project นี้เเบ่งเป็น 3 เเหล่งโดยเเต่ละเเหล่งจะมีข้อมูลเเบบ time stamp 15 min เเละ 1 hr

 - CAMS (มี 2 ประเภทคือ Reanalysis เเละ Forecast)
    - cams_forecast
      - all_site_15min
      - all_site_1hr
    - cams_time_series_15min
    - cams_time_series_1hr
 - DeDe_49stations
   - merge_15min_all49sites
   - DeDe_49sites_metadata.csv
 - ERA5 (มี 2 ประเภทคือ cloud เเละ radiation)
   - cloud
     - all_site_15min
   - radiation
     - all_site_15min
     - all_site_1hr
## Figure
ใส่ภาพกราฟต่างๆ ที่ประมวลผลมาเเล้ว
## merged_data
เก็บไฟล์ CSV ต่างๆ ที่ merge เเล้วนำมาเเยกประเภทดังนี้
- cams_forecast_15min
- cams_forecast_1hr
- era5_cloud_15min
- solar_radiation_15min
- solar_radiation_1hr
## Notebooks

1. **`cams_download.ipynb`**
   
   - download radiation data from **CAMS** using API

2. **`cams_proceed_data.ipynb`**

3. **`era5_proceed_data.ipynb`**

4. **`cams_forecast_proceed_data.ipynb`**

5. **`merge_data.ipynb`**

6. **`matrix_score.ipynb`**

7. **`solar_radiation_visualization.ipynb`**

8. **`cloud_visualization.ipynb`**

9.  **`forecast_today.ipynb`**

---

## Data Folders

- `data`:  

- `merged_data`: 

## Document

report : [Comparison of Modeled Global Horizontal Irradiance (GHI) and Surface Solar Radiation Downwards (SSRD) with Ground-Based Observations
](https://docs.google.com/document/d/1rNs3p3uPPQMf2_isUCiYHSzBlVQZv1ZRGF89WAkM6js/edit?usp=sharing)