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


## RAW Code :

# Import libraries
import requests
import pandas as pd

# Set your GitHub token
api_token = '#personal token'  # Securely stores your token

# GitHub API URL
base_url = 'https://api.github.com'
headers = {'Authorization': f'token {api_token}'}

# Step 2: Scrape Users in Tokyo with over 200 followers
users_data = []
page = 1

while True:
    users_url = f"{base_url}/search/users?q=location:Tokyo+followers:>200&page={page}&per_page=100"
    response = requests.get(users_url, headers=headers)
    data = response.json()
    if 'items' not in data or not data['items']:
        break
    users_data.extend(data['items'])
    page += 1

# Step 3: Extract User Info
users = []
for user in users_data:
    user_detail_url = user['url']
    user_response = requests.get(user_detail_url, headers=headers)
    user_info = user_response.json()

    # Clean up company name
    company = user_info.get('company', '')
    if company:
        company = company.strip().lstrip('@').upper()

    users.append({
        'login': user_info['login'],
        'name': user_info['name'],
        'company': company,
        'location': user_info['location'],
        'email': user_info['email'] or '',  # Handle null emails
        'hireable': 'true' if user_info['hireable'] else 'false',
        'bio': user_info['bio'] or '',  # Handle null bios
        'public_repos': user_info['public_repos'],
        'followers': user_info['followers'],
        'following': user_info['following'],
        'created_at': user_info['created_at']
    })

# Convert to DataFrame and save as CSV
users_df = pd.DataFrame(users)
users_df.to_csv('users.csv', index=False)

# Step 4: Fetch Repositories for Each User
repos = []
for user in users:
    page = 1
    user_repos = []
    while True:
        repos_url = f"{base_url}/users/{user['login']}/repos?sort=pushed&direction=desc&page={page}&per_page=100"
        repos_response = requests.get(repos_url, headers=headers)
        repos_data = repos_response.json()
        if not repos_data or len(user_repos) >= 500:
            break
        for repo in repos_data:
            if len(user_repos) >= 500:
                break
            user_repos.append({
                'login': user['login'],
                'full_name': repo['full_name'],
                'created_at': repo['created_at'],
                'stargazers_count': repo['stargazers_count'],
                'watchers_count': repo['watchers_count'],
                'language': repo['language'],
                'has_projects': 'true' if repo['has_projects'] else 'false',
                'has_wiki': 'true' if repo['has_wiki'] else 'false',
                'license_name': repo['license']['name'] if repo['license'] else ''
            })
        page += 1
    repos.extend(user_repos)

# Convert to DataFrame and save as CSV
repos_df = pd.DataFrame(repos)
repos_df.to_csv('repositories.csv', index=False)

print("Data scraping and file creation completed.")
print(f"Users DataFrame shape: {users_df.shape}")
print(f"Repositories DataFrame shape: {repos_df.shape}")


# code for Q1 to Q16

import pandas as pd
import numpy as np
from scipy.stats import pearsonr, linregress

# Load data from CSV files
users_df = pd.read_csv('users.csv')
repos_df = pd.read_csv('repositories.csv')

# Convert 'created_at' columns to datetime
users_df['created_at'] = pd.to_datetime(users_df['created_at'], errors='coerce')
repos_df['created_at'] = pd.to_datetime(repos_df['created_at'], errors='coerce')

# Question 1: Top 5 users in Tokyo with the highest number of followers
top_5_followers = users_df.nlargest(5, 'followers')['login'].tolist()
print("1. Top 5 users in Tokyo with the highest number of followers:")
print(', '.join(top_5_followers))

# Question 2: 5 earliest registered GitHub users in Tokyo
earliest_5_users = users_df.nsmallest(5, 'created_at')['login'].tolist()
print("2. 5 earliest registered GitHub users in Tokyo:")
print(', '.join(earliest_5_users))

# Question 3: 3 most popular licenses among these users (ignoring missing licenses)
popular_licenses = repos_df['license_name'].value_counts().drop('', errors='ignore').nlargest(3).index.tolist()
print("3. 3 most popular licenses among these users:")
print(', '.join(popular_licenses))

# Question 4: Company where the majority of these developers work
most_common_company = users_df['company'].mode().values[0] if not users_df['company'].isnull().all() else 'None'
print("4. Company where the majority of these developers work:")
print(most_common_company)

# Question 5: Most popular programming language among these users
most_popular_language = repos_df['language'].value_counts().idxmax()
print("5. Most popular programming language among these users:")
print(most_popular_language)

# Question 6: Second most popular programming language among users who joined after 2020
users_post_2020 = users_df[users_df['created_at'] > '2020-01-01']
repos_post_2020 = repos_df[repos_df['login'].isin(users_post_2020['login'])]
second_most_popular_language_post_2020 = repos_post_2020['language'].value_counts().nlargest(2).index[1]
print("6. Second most popular programming language among users who joined after 2020:")
print(second_most_popular_language_post_2020)

# Question 7: Language with the highest average number of stars per repository
average_stars_by_language = repos_df.groupby('language')['stargazers_count'].mean()
highest_avg_star_language = average_stars_by_language.idxmax()
print("7. Language with the highest average number of stars per repository:")
print(highest_avg_star_language)

# Question 8: Top 5 users in terms of leader_strength
users_df['leader_strength'] = users_df['followers'] / (1 + users_df['following'])
top_5_leader_strength = users_df.nlargest(5, 'leader_strength')['login'].tolist()
print("8. Top 5 users in terms of leader_strength:")
print(', '.join(top_5_leader_strength))

# Question 9: Correlation between followers and public repositories
followers_repos_corr, _ = pearsonr(users_df['followers'], users_df['public_repos'])
print("9. Correlation between followers and public repositories:")
print(f"{followers_repos_corr:.3f}")

# Question 10: Regression slope of followers on public repositories
slope, _, _, _, _ = linregress(users_df['public_repos'], users_df['followers'])
print("10. Regression slope of followers on public repositories:")
print(f"{slope:.3f}")

# Question 11: Correlation between projects and wiki enabled
repos_df['has_projects'] = repos_df['has_projects'].fillna(False).astype(bool).astype(int)
repos_df['has_wiki'] = repos_df['has_wiki'].fillna(False).astype(bool).astype(int)

projects_wiki_corr, _ = pearsonr(repos_df['has_projects'], repos_df['has_wiki'])
print("11. Correlation between projects and wiki enabled:")
print(f"{projects_wiki_corr:.3f}")

# Question 12: Difference in average following between hireable and non-hireable users
users_df['hireable'] = users_df['hireable'].fillna(False).astype(bool)
avg_following_hireable = users_df[users_df['hireable']]['following'].mean()
avg_following_non_hireable = users_df[~users_df['hireable']]['following'].mean()
following_diff = avg_following_hireable - avg_following_non_hireable

print("12. Difference in average following between hireable and non-hireable users:")
print(f"{following_diff:.3f}")

# Question 13: Regression slope of followers on bio word count
users_df['bio_word_count'] = users_df['bio'].fillna('').str.split().str.len()
bio_users_df = users_df[users_df['bio_word_count'] > 0]

slope, _, _, _, _ = linregress(bio_users_df['bio_word_count'], bio_users_df['followers'])
print("13. Regression slope of followers on bio word count:")
print(f"{slope:.3f}")

# Question 14: Top 5 users with most repositories created on weekends
repos_df['day_of_week'] = repos_df['created_at'].dt.dayofweek
weekend_repos_df = repos_df[repos_df['day_of_week'].isin([5, 6])]
top_5_weekend_creators = weekend_repos_df['login'].value_counts().nlargest(5).index.tolist()
print("14. Top 5 users with most repositories created on weekends:")
print(', '.join(top_5_weekend_creators))

# Question 15: Fraction of users with email for hireable vs. non-hireable users
users_df['has_email'] = users_df['email'].notna()
hireable_email_fraction = users_df[users_df['hireable']]['has_email'].mean()
non_hireable_email_fraction = users_df[~users_df['hireable']]['has_email'].mean()

email_share_diff = hireable_email_fraction - non_hireable_email_fraction
print("15. Difference in email sharing between hireable and non-hireable users:")
print(f"{email_share_diff:.3f}")

# Question 16: Most common surname
users_df['surname'] = users_df['name'].dropna().str.strip().str.split().str[-1]
surname_counts = users_df['surname'].value_counts()
max_surname_count = surname_counts.max()
most_common_surnames = surname_counts[surname_counts == max_surname_count].index.tolist()

print("16. Most common surname(s):")
print(', '.join(sorted(most_common_surnames)))
print(f"Number of users with the most common surname: {max_surname_count}")



