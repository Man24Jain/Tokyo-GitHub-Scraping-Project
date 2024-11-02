# Tokyo-GitHub-Scraping-Project
"Scraping Tokyo GitHub users with over 200 followers"

 Key Insights
- Data Collection: Scraped GitHub data for users in Tokyo with over 200 followers and analyzed their public repositories.
- **Interesting Finding**: While some developers with detailed bios tend to have more followers, the correlation was analyzed to understand the extent of this relationship.
- **Recommendation**: Developers may consider including detailed and meaningful bios to potentially increase their visibility and followers on GitHub.

## Project Overview
This project involves scraping GitHub data using the GitHub API for users in Tokyo with over 200 followers and their public repositories. The analysis aims to derive meaningful insights from user data, including their bio information, repository statistics, and user engagement patterns.

### Files in This Repository
- **`users.csv`**: Contains user information with the following columns:
  - `login`: GitHub user ID
  - `name`: Full name
  - `company`: Cleaned company name (whitespace trimmed, leading `@` removed, uppercase)
  - `location`: User's location
  - `email`: Email address (empty string if null)
  - `hireable`: Indicates if the user is hireable (`true`/`false`)
  - `bio`: Short bio (empty string if null)
  - `public_repos`: Number of public repositories
  - `followers`: Number of followers
  - `following`: Number of people the user is following
  - `created_at`: Date the user joined GitHub

- **`repositories.csv`**: Contains public repository data for users in `users.csv` with the following columns:
  - `login`: GitHub user ID of the owner
  - `full_name`: Full repository name
  - `created_at`: Date the repository was created
  - `stargazers_count`: Number of stars
  - `watchers_count`: Number of watchers
  - `language`: Main programming language
  - `has_projects`: If the repository has projects enabled (`true`/`false`)
  - `has_wiki`: If the repository has a wiki (`true`/`false`)
  - `license_name`: License name (empty if not available)

## Data Collection Process
1. **API Scraping**: The GitHub API was used to fetch users from Tokyo with more than 200 followers. For each user, up to 500 of their most recently pushed repositories were collected.
2. **Data Cleaning**:
   - Company names were trimmed, leading `@` symbols removed, and converted to uppercase.
   - Null or missing values were standardized to empty strings where necessary.
3. **File Output**:
   - Two CSV files were generated (`users.csv` and `repositories.csv`), preserving the exact API response values for consistency.

## Analysis Highlights
### 1. Bio Length and Follower Correlation
**Objective**: Analyze if having a longer bio helps attract more followers.
- **Method**: Calculated the word count of each bio and performed a linear regression to find the slope between bio word count and number of followers.
- **Result**: The regression slope provides insight into whether there is a positive or negative relationship between bio length and followers.

### 2. Top Repository Creators on Weekends
- **Finding**: Identified which users are most active in creating repositories over weekends (UTC).

### 3. Hireable Users and Email Sharing
- **Analysis**: Compared the fraction of hireable users who share their email addresses versus those who do not.

### 4. Most Common Surname Analysis
- **Method**: Extracted and analyzed the last word of user names as potential surnames to identify the most common one.

## Tools and Technologies Used
- **Programming Languages**: Python
- **Libraries**: `pandas`, `requests`, `scikit-learn`
- **Environment**: Google Colab, Mac M1 Air

## Future Recommendations
- **Developers**: Consider writing detailed bios to increase engagement.
- **Further Analysis**: Explore additional correlations like repository stars and follower count to understand user popularity better.

## How to Run the Code
1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/your-repo.git
   cd your-repo
