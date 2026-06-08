# sentiment_over_time_analysis

This analysis is meant to see how player sentiment changes over time, and how quickly it degrades when a player is going through a slump. I am specifically focusing on hitters to keep the scope manageable and will be using NLP with YouTube and Reddit comments to quantify player sentiment.

There are a few questions I want to answer through this analysis, which I will probably add to as I progress. For right now, I want to know: 1. Do star hitters receive criticism faster?, 2. Which fanbases are most supportive of their players?, and 3. How much does a player's performance need to decline before fan sentiment noticeably drops?

I am planning on getting my hitting data through pybaseball, where I will pay special attention to OPS. I plan on calculating rolling stats, with each weeks OPS being compared to the season OPS to determine slump severity. I then plan on using the YouTube Data API and getting the comment threads from different recap and highlight videos posted daily by the MLB account. I will then use Reddit's API to do a similar thing, filtering by posts on the baseball and team subreddits.

For the sentiment analysis, I am planning on using VADER to classify the comments as positive/negative. I am very inexperienced with this in particular, so I will be using documentation and AI to help guide me through this. Generally, I want to be pretty strict about what counts as positive/negative and will make sure to review any baseball specific slang that might be misinterpreted.

After this is done, I will aggregate per player per week (player-weeks) and exclude any player-weeks with less than 20 comments (might have to adjust this). I will then merge my two datasets (sentiment data and hitting data) to be able to perform my main analysis. I am thinking that I will make categories for slump severity and use that to compare average sentiment per slump category.

Important Visualizations to try: Sentiment vs. slump severity scatterplot, average sentiment by slump category, player specific time series showing OPS vs Sentiment, Team level fan "patience", maybe Reddit vs. YouTube comparison?

Inclusion requirement: Hitters with 100+ plate appearances for 2026 and at least 100 comments.

Outline from ChatGPT:

1. Define the project question

Use something like:

How much does a hitter’s performance need to decline before fan sentiment noticeably drops?

Your unit of analysis should probably be:

player-week

Example:

Aaron Judge | Week of May 12 | rolling OPS | sentiment score | comment count

That keeps the project manageable.

⸻

2. Define your player pool

Start with MLB hitters only.

Possible inclusion rule:

All active MLB position players with at least X plate appearances and at least Y comments.

I would avoid using “active roster” alone because active rosters change constantly.

Better options:

2026 hitters with 100+ PA

or

Players who appeared for an MLB team during the selected date range

That gives you a cleaner, reproducible sample.

⸻

3. Pull baseball performance data

Use pybaseball.

It can retrieve Statcast, FanGraphs, Baseball Reference, batting stats, pitching stats, standings, and player/team data. ￼

You want daily or game-level batting data if possible.

For each hitter, create:

player_id
player_name
team
date
PA
AB
H
BB
1B
2B
3B
HR
SO
OPS
wOBA if available

Then calculate rolling stats:

7-day OPS
14-day OPS
30-day OPS
season-to-date OPS

For this project, I’d start with:

14-day OPS vs season-to-date OPS

That gives a reasonable “slump” measure.

⸻

4. Define “slump severity”

Create a column like:

slump_index = rolling_14_day_OPS - season_to_date_OPS

Or:

slump_pct = rolling_14_day_OPS / season_to_date_OPS - 1

Example interpretation:

-0.25 = player is performing 25% below his season baseline

This becomes your main explanatory variable.

⸻

5. Pull YouTube data

Use the YouTube Data API.

YouTube’s commentThreads.list endpoint returns comment threads for a video and costs 1 quota unit per call. ￼

You do not need to manually collect video IDs one by one.

Workflow:

For each player:
Search YouTube for "[player name] [team] highlights"
Save candidate videos
Filter videos by date, title, channel, view count, comment count
Pull comments from selected videos

Useful fields to save:

source = youtube
player_search_name
video_id
video_title
video_date
channel_title
comment_id
comment_text
comment_date
like_count

Important: keep both video_date and comment_date.

For sentiment timing, comment_date is usually more important.

⸻

6. Pull Reddit data

Use Reddit’s API through PRAW.

PRAW is the common Python wrapper for Reddit’s API and lets you search subreddits, access submissions, and pull comments. ￼

Workflow:

For each player:
Search r/baseball and team subreddit for player name
Save matching posts
Pull comments from those posts

Useful fields:

source = reddit
player_search_name
subreddit
post_id
post_title
post_date
comment_id
comment_text
comment_date
score

Team subreddits are especially useful because they give you fanbase context.

⸻

7. Save raw data first

Do not start cleaning immediately.

Save raw files:

data/raw/youtube_comments_raw.csv
data/raw/reddit_comments_raw.csv
data/raw/player_performance_raw.csv

Then create cleaned versions:

data/processed/comments_clean.csv
data/processed/player_week_sentiment.csv
data/processed/player_week_performance.csv
data/processed/final_player_week_dataset.csv

Raw data protects you when your cleaning logic changes.

⸻

8. Clean comments

Basic cleaning:

remove deleted/removed comments
remove empty comments
remove duplicate comments
remove bot-like comments
normalize dates
remove comments not in English, if needed

But keep a copy of the original text.

Use columns like:

comment_text_raw
comment_text_clean

⸻

9. Match comments to players

This is one of the biggest steps.

Start simple:

If comment contains player last name or full name, assign to that player.

But watch for name ambiguity:

Will Smith
Matt Chapman
Rogers
Martinez
Smith

For ambiguous names, require:

full name
or last name + team context
or video/post title clearly about that player

Create a confidence column:

player_match_confidence = high / medium / low

For your first analysis, only use high.

⸻

10. Run sentiment analysis

Start with something simple:

VADER

Then maybe compare with a transformer model later.

Create:

sentiment_score
sentiment_label

Example labels:

positive: score >= 0.05
neutral: -0.05 < score < 0.05
negative: score <= -0.05

But for sports slang, manually inspect samples.

Words like “filthy,” “nasty,” “sick,” and “disgusting” can be positive in baseball.

⸻

11. Aggregate to player-week level

This is where it becomes analyzable.

Group by:

player
team
week
source

Calculate:

comment_count
avg_sentiment
negative_pct
positive_pct
neutral_pct
avg_engagement

Then apply your inclusion rule:

Only include player-weeks with at least 20 comments

Or for players overall:

Only include players with at least 100 total comments

This prevents one random angry comment from driving results.

⸻

12. Merge with performance data

Now join:

player_week_sentiment

- player_week_performance

Final dataset:

player
team
week
source
comment_count
avg_sentiment
negative_pct
rolling_14_day_OPS
season_to_date_OPS
slump_pct

Now your project is finally ready for analysis.

⸻

13. Main analysis

Start with simple questions:

Does sentiment decline when slump_pct gets worse?

Then:

How large does the slump need to be before sentiment noticeably drops?

You could bucket slump severity:

normal: -10% to +10%
mild slump: -10% to -25%
major slump: -25% to -40%
severe slump: below -40%

Then compare average sentiment by bucket.

⸻

14. Compare Reddit vs YouTube

This is a great side analysis.

Compare:

comment volume
negative_pct
avg_sentiment
vulgarity/toxicity
player match rate
spam rate
missingness
date coverage

Possible result:

Reddit is noisier but more specific.
YouTube is larger but less analytically clean.

That itself is a useful finding.

⸻

15. Visualizations

I’d make these:

Sentiment vs slump severity scatterplot
Average sentiment by slump bucket
Player-specific timeline: OPS vs sentiment
Team-level patience index
Reddit vs YouTube comparison chart

The best “hero graphic” is probably:

How far can performance fall before sentiment turns negative?

⸻

16. Create a “Patience Index”

Something like:

Patience Index = average slump_pct when sentiment first drops below player baseline

Or simpler:

Patience Index = sentiment drop per 10% OPS decline

Interpretation:

Larger negative value = fans turn faster
Smaller negative value = fans are more patient

⸻

17. Final write-up structure

Use this structure:

1. Why fan patience is worth measuring
2. Data sources: Reddit, YouTube, batting performance
3. How comments were matched to players
4. How sentiment was measured
5. How slumps were defined
6. Main findings
7. Reddit vs YouTube data quality
8. Limitations
9. What I would improve next

⸻

18. Biggest risks to watch for

The main ones:

Name ambiguity
Small sample sizes
Sports slang confusing sentiment models
Fans reacting to one viral moment, not actual performance
Comments about the team, not the player
YouTube spam
Reddit sarcasm
Star expectations vs role-player expectations

Do not hide these. Put them in the limitations section. That makes the project look more mature.

⸻

My suggested next move: build a tiny proof-of-concept with 5 hitters, both sources, and one month of data. Once that works, scaling up is mostly repetition.
