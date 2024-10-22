## Analysis and Trend Evaluation of Medical Device Recalls

### Table of Contents
1. [Project Overview](#project-overview)
2. [Project Objective](#project-objective)
3. [Applicable Stakeholders](#applicable-stakeholders)
4. [Data Sources](#data-sources)
5. [Methodology](#methodology)
6. [Results](#results)
    - [Frequency Analysis](#frequency-analysis)
    - [Geographical and Manufacturer Analysis](#geographical-and-manufacturer-analysis)
7. [Contributions](#contributions)
8. [Usage](#usage)
9. [Dependencies](#dependencies)
10. [Contact](#contact)
11. [License](#license)

### Project Overview
Manufacturers are not always able to catch defects or adverse effects during the lifecycle of a medical device, from raw material sourcing to patient use. Many issues, such as design flaws, manufacturing errors, or labeling inconsistencies, can go unnoticed until devices are already being utilized, leading to recalls and safety alerts. Using data from the ICIJ’s International Medical Devices Database (IMDD), which contains over 120,000 records of recalls and safety notices worldwide, we analyze these common causes to reveal patterns of frequent or severe recalls across regions, device categories, and manufacturers. This analysis helps drive improvements in regulatory oversight and the development of safer, more reliable medical devices.

### Project Objective
The purpose of this study is to analyze medical device recall trends and identify the most common causes, such as design flaws, manufacturing errors, and labeling issues. By comparing recall rates across different regions, device categories, and manufacturers, the study aims to uncover patterns that contribute to frequent or severe recalls. This research intends to highlight areas where improvements in device development and oversight can reduce patient risks and enhance overall medical device safety throughout their lifecycle.

### Applicable Stakeholders
The findings from this data analysis are intended to inform key stakeholders, including:
- Regulatory bodies
- Original Equipment Manufacturers (OEMs)
- Contract manufacturers
- Healthcare providers
- Patients directly affected by the recalls

### Data Sources
We leveraged publicly available datasets from the ICIJ's International Medical Devices Database (IMDD):

| File Name                         | Description                                                                                                                | Size  | URL                                          |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------------|-------|----------------------------------------------|
| **IMDD events-1681209680.csv**   | Details specific actions or incidents related to medical devices, including action type, cause, status, and geographical info. | 68.7 MB | [Link](https://medicaldevices.icij.org/p/about) |
| **IMDD devices-1681209661.csv**  | Provides comprehensive details about the medical devices including name, description, classification, and risk level.          | 149.2 MB | [Link](https://medicaldevices.icij.org/p/about) |
| **IMDD manufacturers-1681209657.csv** | Details information about companies producing medical devices including company name, address, and representative contacts.   | 9.5 MB | [Link](https://medicaldevices.icij.org/p/about) |

### Methodology

#### Data Manipulation and Cleaning
To preprocess the data from the IMDD dataset, Python was used within Jupyter notebooks to load CSV files and convert them into pandas DataFrames for further manipulation and analysis.

![image](https://github.com/user-attachments/assets/82b770b1-318f-403b-8374-26f771d744dc)

1. **Import CSVs**:
    ```python
    import pandas as pd
    events_df = pd.read_csv('path/to/events-1681209680.csv', low_memory=False)
    devices_df = pd.read_csv('path/to/devices-1681209661.csv', low_memory=False)
    manufacturers_df = pd.read_csv('path/to/manufacturers-1681209657.csv', low_memory=False)
    ```

2. **Merge Datasets**:
    ```python
    merged_df1 = pd.merge(events_df, devices_df, left_on='device_id', right_on='id', how='left')
    final_df = pd.merge(merged_df1, manufacturers_df, left_on='manufacturer_id', right_on='id', how='left')
    ```

3. **Initial Data Cleaning**:
    ```python
    final_df.drop_duplicates(inplace=True)
    final_df['date'] = pd.to_datetime(final_df['date'], errors='coerce')
    final_df.columns = final_df.columns.str.lower().str.replace(' ', '_')
    ```

4. **Handling Missing Values**:
    ```python
    essential_columns = ['device_id', 'manufacturer_id', 'date', 'reason']
    final_df.dropna(subset=essential_columns, inplace=True)
    important_categorical_cols = ['classification', 'country_x', 'determined_cause', 'description', 'name_y']
    for col in important_categorical_cols:
        if col in final_df.columns:
            mode_value = final_df[col].mode()
            if not mode_value.empty:
                final_df[col].fillna(mode_value[0], inplace=True)
    columns_to_drop = [
        'action', 'action_classification', 'action_level', 'data_notes',
        'date_initiated_by_firm', 'date_posted', 'date_terminated', 'date_updated',
        'documents', 'address', 'comment', 'representative', 'create_date'
    ]
    final_df.drop(columns=columns_to_drop, inplace=True)
    final_df.fillna({
        'target_audience': 'Unknown',
        'authorities_link': 'N/A',
        'status': 'Unknown',
        'distributed_to': 'N/A',
        'implanted': 'N/A',
        'number_x': 'N/A',
        'number_y': 'N/A',
        'quantity_in_commerce': 'N/A',
        'risk_class': 'N/A',     
        'action_summary': 'No Summary', 
        'uid': 'N/A',                    
        'uid_hash': 'N/A',              
        'url': 'No URL',                 
        'name_x': 'Unknown Device Name', 
        'name_y': 'Unknown Manufacturer Name', 
        'description': 'No Description',       
        'code': 'Unknown',                
        'determined_cause': 'Unknown Cause', 
        'parent_company': 'Unknown Parent Company' 
    }, inplace=True)
    ```

5. **Final DataFrame Validation**:
    ```python
    file_path = 'path/to/dataset_merged.xlsx'
    final_df.to_excel(file_path, index=False, engine='xlsxwriter')
    ```

### Results

#### Frequency Analysis
The primary objective of this analysis is to explore and understand the common causes of medical device recalls.

1. **Truncated Long Texts**:
    ```python
    df['reason'] = df['reason'].apply(lambda x: x if len(x) < 100 else x[:100] + '...')
    ```

2. **Frequency and Mode Analysis**:
    ```python
    recall_cause_mode = df['reason'].mode()[0]
    recall_cause_freq = df['reason'].value_counts()
    top_n = 20
    top_recall_cause_counts = recall_cause_freq.head(top_n)
    ```

3. **Visualization**:
    ```python
    # Horizontal Bar Chart
    import seaborn as sns
    import matplotlib.pyplot as plt

    colors = sns.color_palette("Blues", len(top_recall_cause_counts))[::-1]
    plt.figure(figsize=(14, 10))
    bar_plot = sns.barplot(x=top_recall_cause_counts.values, y=top_recall_cause_counts.index, palette=colors)

    for index, value in enumerate(top_recall_cause_counts.values):
        plt.text(value, index, f' {value}', va='center', ha='left', fontsize=12)

    plt.title('Most Frequent Causes of Medical Device Recalls', fontsize=20)
    plt.xlabel('Frequency', fontsize=18)
    plt.ylabel('Recall Cause', fontsize=18)
    plt.xticks(fontsize=14)
    plt.yticks(fontsize=14)
    plt.show()

    # Treemap
    import plotly.graph_objects as go

    labels = ["Device failure before use", "Device failure after use", "Other",
              "Manufacturing", "Material / Component", "Packing - Sterility", "Packing - Labeling", "Design", "Software", "Labeling", "Instructions", "Mechanical failure", "Other failure", "Operating instructions failure", "Electronic failure", "General Other", "Operating error", "MDA Safety Information", "Nonoxinol-9", "GMP", "Contamination"]
    parents = ["", "", "",
               "Device failure before use", "Device failure before use", "Device failure before use", "Device failure before use", "Device failure before use", "Device failure before use", "Device failure before use", "Device failure before use",
               "Device failure after use", "Device failure after use", "Device failure before use", "Device failure after use",
               "Other", "Other", "Other", "Other", "Other", "Other"]
    values = [1940, 124, 82,
              748, 419, 241, 159, 133, 120, 88, 43,
              45, 40, 29, 19,
              15, 13, 12, 12, 10, 10]

    fig = go.Figure(go.Treemap(
        labels=labels,
        parents=parents,
        values=values,
        textinfo="label+value+percent entry",
        hoverinfo="label+value+percent entry",
        marker=dict(colors=values, colorscale='Blues'),
        textfont=dict(size=20) 
    ))

    fig.update_layout(
        title='Medical Device Recall Causes Treemap',
        title_font_size=24,
        autosize=True,
        width=1400, 
        height=900, 
        margin=dict(t=50, l=25, r=25, b=25),
        treemapcolorway=["#4477AA", "#CCBB44", "#DDCC77"],
    )
    fig.show()
    ```

### Key Findings

1. **Top Recall Causes**:
    - Manufacturing defects leading to device failure before use were identified as the most common recall cause.
    - Other significant causes include failures related to materials/components, packaging sterility, labeling, and design issues.

2. **Geographical Insights**:
    - Major recalls concentrated in the United States, Canada, and Australia.
    - Analysis excluding the major outliers provided a clearer global recall pattern.

3. **Manufacturer Trends**:
    - Top companies with highest recalls included Siemens AG (3,071 recalls), Stryker, and Johnson & Johnson.

### Contributions

- **Hunter Belous**: Responsible for visualizations in the "Medical Device Recalls by Parent Company" section, introduction, data sources, and initial manipulation slides. Uncovered key trends among parent companies.
- **Jared Fox**: Created visualizations in the "Medical Device Recalls by Region" section and the summary slide. Played a key role in refining the dataset for enhanced visualization.
- **Asad Kamal**: Developed visualizations for the "Frequency Analysis" and "Insights from Device Failure Analysis" sections. Authored the dataset merging and cleaning notebook.

### Usage

To replicate or build upon this analysis:

1. **Clone the Repository**:
    ```sh
    git clone https://github.com/akamal341/medical-device-recall-analysis.git
    cd medical-device-recall-analysis
    ```

2. **Install Dependencies**:
    Ensure you have the required Python libraries:
    ```sh
    pip install -r requirements.txt
    ```

3. **Run Jupyter Notebooks**:
    Open and run the provided Jupyter Notebooks to see the data manipulation and visualizations.

### Dependencies

Dependencies for this project include:
- Python 3.9
- Pandas 1.5.3
- NumPy 1.24.1
- Matplotlib 3.7.2
- Seaborn 0.12.2
- Jupyter Notebook
- Squarify
- Plotly

### Contact

For any questions or contributions, please contact:
- **Asad Kamal** - aakamal {/@/} umich {/dot/} edu
