# job-webscraper

LLM API-Powered Job Listings Data Cleaning Tool: Showcase for Data Scientists

This project leverages Large Language Model (LLM) API technology to automate the cleaning and transformation of job listings for data scientists. It filters postings based on criteria like location and salary, using LLM APIs to enhance data accuracy and usability, streamlining the job search process. Explore how AI revolutionizes job data handling for more efficient, targeted career advancement.



# 1. Define the Problem

### Objective
Showcase the application of Large Language Model (LLM) API technology in cleaning and transforming job listings data to highlight the most relevant opportunities for data scientists. This project aims to demonstrate the power of LLM APIs in automating the extraction, cleaning, and enrichment of job postings from various websites based on specified criteria such as location, salary, company size, and preferred tech stack, thereby streamlining the job search process for data science professionals.

### Key Inquiries
- Which industry demonstrates the highest demand for Data Scientists?
- Which industry offers the most competitive average salary?
- Which industries mandate educational qualifications beyond a bachelor's degree?
- Which industries predominantly require on-site presence, and which ones permit remote work?
- What correlation exists between salary levels and remote work availability?



# 2. Data Collection

### Data Sources
- **SerpAPI**: Leverages the "google_jobs" engine to fetch relevant job listings.
- **Documentation URL**: [SerpAPI Google Jobs API](https://serpapi.com/google-jobs-api)

### Reason for Selecting SerpAPI
The choice of SerpAPI is driven by its capability to source job listings from a multitude of platforms, coupled with its offer of a free tier for API queries.

### Implementation of a Custom Job Search Enhancement
To address the limitation of receiving only 10 job results per API call within the free tier of SerpAPI's Google Jobs engine, a specialized Python class has been developed. This class is capable of fetching additional pages, thereby significantly increasing the volume of unique job listings obtained.

[IMAGE HERE] - SEARCH FUNCTION WORKFLOW

### Data Dictionary - Raw Data

| Column Name    | Description                                                                 | Data Type     | Example                                              |
|----------------|-----------------------------------------------------------------------------|---------------|------------------------------------------------------|
| id             | Unique identifier for each job listing.                                     | Integer       | 1                                                    |
| title          | Title of the job listing.                                                   | String        | "Staff Data Scientist"                               |
| company_name   | Name of the company offering the job.                                       | String        | "GOBankingRates"                                     |
| location       | Geographic location where the job is based.                                 | String        | "Anywhere" or "Los Angeles, CA"                      |
| via            | Source platform or website via which the job listing was found.             | String        | "via LinkedIn"                                       |
| job_link_url   | URL to the job listing on the source platform or website.                   | String        | A Google search URL pointing to job listing details. |
| job_link_text  | Text associated with the job listing link.                                  | String        | "See web results for GOBankingRates"                 |
| description    | Brief description of the job listing.                                      | String        | "GOBankingRates™ is unique in the digital marketing..." |
| extensions     | Additional details about the job listing such as date posted, employment type, and benefits. | String | "1 day ago--Work from home--Full-time--Health insurance" |
| job_id         | Encoded identifier for the job listing, potentially used by the API.        | String        | An encoded string representing the job's unique ID.  |
| api_status     | Status of the job listing retrieval via API.                                | String        | "Success"                                            |
| api_runtime    | Timestamp indicating when the job listing was retrieved via API.            | String        | "2024-02-28 08:35:53+00:00" (ISO 8601 format)        |
| engine         | The search engine used for retrieving job listings.                         | String        | "google_jobs"                                        |
| google_domain  | Google domain on which the job search was performed.                        | String        | "google.com"                                         |
| query          | Search query used to find the job listings.                                 | String        | "Data Scientist Jobs in Los Angeles"                 |


## High-Level EDA on Job Listings Dataset

![Data Preparation - Raw Overview](https://github.com/Prvargas/job-webscraper/blob/main/img/A%20-%20Dataprep_Raw_Overview.png)

### Dataset Overview
- **Total Entries**: 100 job listings.
- **Columns**: 14 fields including `title`, `company_name`, `location`, `description`, `salary_range`, and more.
- **Unique Job Titles**: 64, with "Data Scientist" being the most frequent.
- **Top Hiring Company**: VirtualVocations, appearing 8 times.
- **Most Common Location**: Los Angeles, CA, with 87 listings.
- **Primary Source**: Listings primarily found via LinkedIn.



![Data Preparation - Raw Variables](https://github.com/Prvargas/job-webscraper/blob/main/img/B%20-%20Dataprep_Raw_Variables.png)




# 3. Data Cleaning and Preprocessing

![Data Preparation - Raw Duplicates](https://github.com/Prvargas/job-webscraper/blob/main/img/C%20-%20Dataprep_Raw_Duplicates.png)


### Handling Duplicates
Initially, the "job_id" column was assessed for identifying duplicate job listings. However, the analysis revealed that the "description" field is more suitable for this task. To address duplicates effectively, a bespoke function was developed specifically for identifying duplicates based on the job description.




# 4. Data Transformation

The OpenAI ChatGPT chat completion API was used to transform the raw job listings data. This approach overcomes max token issues as well as memory context loss issues.


[IMAGE HERE] - TRANSFORMATION FUNCTION WORKFLOW


### PROMPT USED IN API CALLS

Only reply in JSON format. 

This is the logic for transforming each column from the original data to the new structure:
- **title**: Check the original title against a predefined list of job titles. If the title is one of "Data Scientist", "Data Analyst", "Machine Learning Engineer", "Data Engineer", "Business Intelligence (BI) Analyst", it remains unchanged; otherwise, it is transformed to "Other".
- **leadership**: Extract from the original title leadership titles like "Team Lead", "Supervisor", "Manager", "Director", "Executive", or "None".
- **focus**: Extract any variations or specializations from the original title that are not part of the standard job titles listed above. If the title is "Marketing Data Analyst", the focus is "Marketing".
- **experience_level**: Analyze the description to determine the experience required:
  - "Entry" for descriptions mentioning 0 to 1 year of experience.
  - "Mid" for descriptions mentioning 2 to 4 years of experience.
  - "Senior" for descriptions mentioning more than 5 years of experience.
- **company_name**: Retain the company names in proper case as they appear.
- **company_industry**: Infer the industry from the company name or description, such as "Retail" for Walmart. For TikTok, which is known for social media, the industry is "Social Media".
- **location**: Use the full location as provided, typically including city and sometimes state or country.
- **city**: Extract the city portion from the location column.
- **state**: Extract the state portion from the location column.
- **remote**: Determine the job work arrangement from the description or extensions, using only "Remote", "On-Site", "Hybrid", or "Ambiguous". If the description mentions a hybrid work schedule, the value is "Hybrid".
- **salary_range**: 
  - Format the salary information into ``"$100k - $130k"`` format or ``"$100k"`` if a single salary is provided. For the given salary range of ``$108,300 to $228,000``, it would be formatted as ``"$108k - $228k"``.
  - If an hourly wage is provided, calculate the annual salary range by multiplying the hourly wage by 40 hours per week, then by 52 weeks per year. Format the result as a range in the format ``"$100k - $130k"``. If the result does not neatly fit into the "k" formatting, round the salary to the nearest thousand and use the appropriate "k" notation.
  - Using the given range of ``$26.20 to $51.39 hourly``, the annual salary range would be approximately ``$54,496 to $106,995``, which would be formatted as ``"$54k - $107k"``.
- **salary_min and salary_max**: Convert the given salary range into integer minimum and maximum values.
- **employment_type**: Extract the employment type from the description or extensions, which should be "Full-Time", "Part-Time", or "Contract". The provided example indicates "Full-Time".
- **required_education**: Determine the minimum required education from the job description. The options are "None", "Bachelors", "Masters", or "Ph.D". Given that the description mentions BS/MS degrees, the required education is "Bachelors".



Example JSON Response:
```JSON
{
  "title": "Data Scientist",
  "leadership": "Supervisor",
  "focus": "Product Analytics",
  "experience_level": "Senior",
  "company_name": "TikTok",
  "company_industry": "Social Media",
  "location": "Los Angeles, CA",
  "city": "Los Angeles",
  "state": "CA",
  "remote": "Hybrid",
  "salary_range": "$108k - $228k",
  "salary_min": 108000,
  "salary_max": 228000,
  "employment_type": "Full-Time",
  "required_education": "Bachelors"
}
```

Use the following job data to create the example JSON:

{Single_Job_Data}



## Data Dictionary - Transformed Data

| Column Name        | Description                                            | Data Type | Example              |
|--------------------|--------------------------------------------------------|-----------|----------------------|
| title              | Job title                                              | String    | Data Scientist       |
| leadership         | Level of leadership role, if any                       | String    | Supervisor           |
| focus              | Area of specialization or focus                        | String    | Product Analytics    |
| experience_level   | Level of experience required                           | String    | Senior               |
| company_name       | Name of the company                                    | String    | TikTok               |
| company_industry   | Industry sector of the company                         | String    | Social Media         |
| location           | Geographic location of the job                         | String    | Los Angeles, CA      |
| city               | City of the job location                               | String    | Los Angeles          |
| state              | State of the job location                              | String    | CA                   |
| remote             | Type of work arrangement (Remote, On-Site, Hybrid)     | String    | Hybrid               |
| salary_range       | Range of salary offered                                | String    | $108k - $228k        |
| salary_min         | Minimum salary offered                                 | Integer   | 108000               |
| salary_max         | Maximum salary offered                                 | Integer   | 228000               |
| employment_type    | Type of employment (Full-Time, Part-Time, Contract)    | String    | Full-Time            |
| required_education | Minimum level of education required                    | String    | Bachelors            |
| job_id             | Job ID to help link to back to raw dataset             | String    | eyJqb2JfdGl0bGU...   |


## High-Level EDA on Transformed Job Listings Dataset

![Data Preparation - Transform Overview](https://github.com/Prvargas/job-webscraper/blob/main/img/D%20-%20Dataprep_Transform_Overview.png)


### Key Takeaways

1. **Data Overview**:
    - The dataset contains job listings with several attributes, including `title`, `leadership`, `focus`, `experience_level`, `company_name`, `company_industry`, `location`, `remote`, `salary_range`, `employment_type`, and `required_education`.
    - There are numerical columns for `salary_min` and `salary_max` which provide the salary range for each job listing.

2. **Missing Values**:
    - Missing values were identified in `salary_min` and `salary_max` (5 entries each), suggesting that not all job listings include salary information. This could impact salary analysis and require imputation or exclusion of these records.
    - Minor missing values in `city` (1 entry) and `state` (2 entries), which may affect location-based analysis but can be easily addressed.

3. **Unique Values and Diversity**:
    - The dataset showcases a diverse range of job titles (7 unique), leadership roles (10 unique), and focuses (68 unique), indicating a wide variety of job listings.
    - Experience levels are categorized into 3 unique values, suggesting a clear segmentation of job roles based on experience.
    - There are 41 unique industries and 10 unique cities listed, demonstrating the dataset's coverage across different sectors and locations.
    - Remote work options are specified with 5 unique values, highlighting varying work arrangement preferences.

4. **Salary Insights**:
    - Salary data shows a wide range, with `salary_min` ranging from `$0 to $200,000` and `salary_max` from `$0 to $500,000`. The presence of `$0` salaries might indicate unpaid roles or missing data that needs further investigation.
    - The average `salary_min` is approximately `$100,648`, and the average `salary_max` is around `$163,651`, suggesting a mid to high salary range for the listed positions.

5. **Data Consistency and Integrity**:
    - All job listings have an employment type of "Full-Time", indicating a lack of part-time, contract, or freelance roles in this dataset.
    - The dataset is consistent in terms of required education, with 4 unique values, ensuring a clear understanding of the educational requirements for the roles listed.

### Impact on Analysis

- **Salary Analysis**: The presence of missing and $0 salary values necessitates careful handling during salary trend analysis. Imputation strategies or exclusion of these records may be required to maintain accuracy.
- **Location-Based Insights**: Missing values in `city` and `state` could slightly affect location-based analysis but are minimal and manageable.
- **Diversity of Job Roles**: The wide variety of job titles, focuses, and industries allows for in-depth analysis of job market trends across different sectors. However, the analysis must account for the varied levels of representation among these categories.
- **Remote Work Trends**: The dataset can provide insights into the prevalence and preference for remote work arrangements across different industries and roles.
- **Educational Requirements**: Analysis of educational requirements across industries and job roles can be conducted, offering insights into the qualifications demanded by employers.


![Data Preparation - Transform Variables](https://github.com/Prvargas/job-webscraper/blob/main/img/E%20-%20Dataprep_Transform_Variables.png)



# 5. Data Visualizations and Insights

### Key Inquiries
- Which industry demonstrates the highest demand for Data Scientists?
- Which industry offers the most competitive average salary?
- Which industries mandate educational qualifications beyond a bachelor's degree?
- Which industries predominantly require on-site presence, and which ones permit remote work?
- What correlation exists between salary levels and remote work availability?



## 5A. Which industry demonstrates the highest demand for Data Scientists??

![Top 5 Industries Hiring the Most Data Scientists](https://github.com/Prvargas/job-webscraper/blob/main/img/01-Top_5_Hiring.png)


The visualization and data indicate that the top 5 industries hiring the most Data Scientists are:
- **Social Media** with 10 job listings
- **Healthcare** with 8 job listings
- **Government** with 6 job listings
- **Consulting** with 5 job listings
- **IT Services** with 4 job listings

These findings reveal the diverse range of industries seeking data science expertise, highlighting the field's broad applicability and demand.

For job seekers in the field of data science, the insight that the **Social Media** industry has the most job listings carries several important implications:

1. **High Demand**: The Social Media industry's position as the top hirer indicates a high demand for data science skills within this sector. Social media companies are likely leveraging data science for a variety of purposes, including user behavior analysis, content recommendation algorithms, advertising effectiveness, and more. This demand suggests robust career opportunities for data scientists interested in these areas.

2. **Skill Relevance**: Job seekers should consider developing or highlighting skills that are particularly relevant to the social media industry. This might include expertise in natural language processing (NLP), machine learning models for recommendation systems, sentiment analysis, big data technologies, and analytics tools. A strong understanding of social media metrics and user engagement strategies could also be advantageous.

3. **Innovative Environment**: The social media sector is known for its fast-paced and innovative environment. Job seekers looking to work on cutting-edge projects and who enjoy a dynamic work setting may find particularly appealing opportunities within this industry.

4. **Portfolio Development**: For those aiming to enter the social media space, developing a portfolio that showcases projects relevant to the industry could be a strategic move. Projects could involve analysis of social media data, development of predictive models based on user data, or exploration of trends and patterns within social networks.

5. **Networking and Learning**: Given the competitive nature of the industry, continuous learning and professional networking are key. Engaging with communities related to data science and social media, attending industry conferences, and participating in relevant online forums can provide insights into industry trends, expand professional networks, and uncover job opportunities.

Overall, the prominence of the Social Media industry as a leading hirer of Data Scientists underscores the sector's reliance on data-driven decision-making and innovation. For job seekers, this highlights the importance of aligning their skills and career aspirations with the needs of this dynamic and influential field.




## 5B. Which industry offers the most competitive average salary?

![Top 5 Industries with Highest Average Minimum Salary](https://github.com/Prvargas/job-webscraper/blob/main/img/02-Top_5_Salary_Min.png)

The horizontal bar chart visualizes the top 5 industries with the highest average minimum salary for Data Scientists, each labeled with the specific average minimum salary value. These industries are:
- **Music** with an average minimum salary of ``$200,000``
- **CRM (Customer Relationship Management)** with an average minimum salary of ``$188,000``
- **Online Dating** with an average minimum salary of ``$185,000``
- **Digital Marketing and Media** with an average minimum salary of ``$180,000``
- **Transportation** with an average minimum salary of ``$150,000``

These findings highlight the industries offering the most competitive starting salaries for Data Scientists, suggesting sectors where the demand for data science skills is particularly high and valued.


For job seekers in the field of data science, the finding that the **Music** industry offers the highest average minimum salary has several key implications:

1. **Valued Expertise**: The Music industry's high compensation levels indicate a strong valuation of data science expertise. This suggests that data scientists can play a crucial role in this sector, likely through analytics and insights that drive strategic decisions, enhance user experience, and optimize content delivery.

2. **Sector-Specific Skills**: To be competitive in the Music industry, job seekers might need to acquire or sharpen specific skills relevant to the sector. This could include knowledge of audio data processing, recommendation systems, user behavior analytics, and digital rights management analytics. Familiarity with the music business model and copyright laws might also be advantageous.

3. **Innovative Opportunities**: High salaries are often tied to roles that involve innovation and critical problem-solving. For job seekers, this suggests that the Music industry may offer unique opportunities to work on innovative projects at the intersection of music, technology, and data science.

4. **Career Growth and Stability**: The high average minimum salary also points to potential for career growth and financial stability within this industry. Job seekers may find long-term career opportunities that not only satisfy their professional aspirations but also offer substantial financial rewards.

5. **Broader Industry Trends**: While the Music industry stands out in terms of salary, this insight can encourage job seekers to look beyond traditional tech sectors for data science roles. Exploring industries with unique applications of data science could uncover unexpected and rewarding career paths.




## 5C. Which industries mandate educational qualifications beyond a bachelor's degree?

![Education Level Requirements Pie Chart](https://github.com/Prvargas/job-webscraper/blob/main/img/04-Education_PieChart.png)

![Education Level Requirements Heatmap](https://github.com/Prvargas/job-webscraper/blob/main/img/03-Education_Heatmap.png)


### CONSULTING
For job seekers aiming at the data science market, the insight that the Consulting industry predominantly requires education levels beyond a Bachelor's degree holds significant implications:

1. **Advanced Education Focus**: The Consulting industry's emphasis on postgraduate education suggests a high value placed on advanced analytical skills, theoretical knowledge, and specialized expertise. This may reflect the industry's need for professionals who can tackle complex problems, provide strategic insights, and implement advanced data-driven solutions for clients.

2. **Preparation for Higher Roles**: For those aspiring to enter the Consulting field, pursuing a Master's or Ph.D. degree can be a strategic move. It not only aligns with the industry's hiring preferences but also prepares candidates for higher-level roles that demand deep analytical prowess and the ability to manage intricate projects.

3. **Competitive Edge**: In a competitive job market, having a postgraduate degree in a relevant field could provide job seekers with a competitive edge. It signifies a commitment to the discipline and a readiness to take on the challenging and high-stakes environment that consulting often presents.

4. **Continual Learning and Specialization**: The preference for higher education levels underscores the importance of continual learning and specialization. Job seekers should focus on areas of data science that are particularly relevant to consulting, such as business analytics, data management, machine learning, and optimization techniques.

5. **Networking and Professional Development**: Engaging with professional networks, attending industry conferences, and participating in workshops can be beneficial. These activities can provide insights into the consulting industry's evolving needs and help build connections that could lead to job opportunities.




## 5D. Which industries predominantly require on-site presence, and which ones permit remote work?


### ON-SITE:

![Top 5 Industries Requiring On-site Work](https://github.com/Prvargas/job-webscraper/blob/main/img/05-Top_5_Onsite.png)


The horizontal bar chart above displays the top 5 industries requiring on-site work for Data Scientists:

- **Government** with 6 job listings
- **Consulting** with 5 job listings
- **Social Media** with 4 job listings
- **Healthcare** with 3 job listings
- **Media** also with 3 job listings

This visualization highlights industries with a strong preference for on-site data science roles, indicating sectors where physical presence and direct collaboration are highly valued. Also, there are likely security concerns regarding roles in Government.


### REMOTE:

![Top 5 Industries Allowing Remote Work](https://github.com/Prvargas/job-webscraper/blob/main/img/06-Top_5_Remote.png)

The horizontal bar chart above illustrates the top 5 industries allowing remote work for Data Scientists:

- **AI (Artificial Intelligence)** with 2 job listings
- **Entertainment** with 1 job listing
- **Technology** with 1 job listing
- **Project Management** with 1 job listing
- **Transportation** also with 1 job listing

This visualization showcases industries more inclined to offer remote work opportunities for data science roles, reflecting a modern approach to employment that values flexibility and acknowledges the effectiveness of remote collaboration in the data science field.




### OVERALL - PIE CHART

![Remote Work Status Pie Chart](https://github.com/Prvargas/job-webscraper/blob/main/img/07-Remote_Status_PieChart.png)


This visualization offers a straightforward snapshot of work arrangement trends in the data science sector, highlighting that a majority (58%) of job listings favor on-site work.




## 5E. What is the relationship between salary and remote work status? 

![Remote Work Salary Box Plot](https://github.com/Prvargas/job-webscraper/blob/main/img/08-Remote_Salary_BoxPlot.png)

The boxplot visualizes the relationship between minimum salary and remote work status for Data Scientist positions. It compares the distribution of minimum salaries across different categories of remote work: On-Site, Remote, Hybrid, and Ambiguous. 

Key observations from the plot include:
- **On-Site** positions tend to have a broad range of minimum salaries, with some roles offering very high starting salaries.
- **Remote** roles also display a wide distribution of minimum salaries, suggesting competitive compensation is available for remote positions.
- **Hybrid** work arrangements show a somewhat tighter distribution of minimum salaries, indicating a consistent range of salary offerings for these types of roles.
- Positions with an **Ambiguous** remote work status seem to have a similar range of minimum salaries compared to the other categories.

This analysis reveals that while there are variations in salary offerings based on remote work status, competitive salaries are present across all categories, underscoring the flexibility in work arrangements without necessarily compromising on compensation.




# 6. Conclusion

Throughout this project, we've conducted a comprehensive analysis of a dataset containing job listings for Data Scientists, revealing valuable insights into the industry's current state, demand across sectors, education requirements, and the evolving landscape of work arrangements. This exploration has underscored the critical role of data science across various industries, from Social Media and Consulting to Healthcare and Government, each with unique demands for expertise and educational qualifications.

## Key findings include:
- The **Social Media** industry emerges as the top hirer, suggesting a vibrant market for data science skills in this sector.
- **Music** and **CRM** industries offer the highest average minimum salaries, indicating areas where data science expertise is highly valued.
- Advanced degrees are particularly prized in sectors like **Consulting**, where a majority of listings require more than a Bachelor's degree.
- While there's a significant preference for on-site work, industries like **AI** and **Technology** are more open to remote arrangements, reflecting a modern approach to work that values flexibility.
- The relationship between salary and remote work demonstrates that competitive compensation is achievable across various work arrangements, challenging the notion that remote or hybrid roles inherently offer lower pay.

The use of Language Model for Learning (LLM) API technology in this project has proven instrumental in streamlining the analysis and interpretation of complex job market data, transforming raw information into actionable insights. For data science professionals navigating the job search process, these findings facilitated by LLM API technology can guide strategic decisions about career development, education paths, and job seeking strategies. By leveraging such advanced technologies, professionals can better align their skills and aspirations with market demands and opportunities, ultimately enhancing their job search efficiency and effectiveness.

In conclusion, the intersection of data science and LLM API technology not only illuminates current job market trends but also exemplifies the power of advanced analytical tools in simplifying complex datasets into meaningful insights. For data science professionals, staying informed about these trends and adapting to the evolving landscape will be key to seizing the best opportunities in a competitive field.

This project highlights the effectiveness of LLM API technology in making the job search process more accessible and informative for data science professionals, pointing towards a future where data-driven decision-making is at the forefront of career advancement strategies.




