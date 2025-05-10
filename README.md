# Dynamic Leaderboard

Speakly test task

---

## Live Demo

[View Live on Netlify](https://magnificent-melba-aacc02.netlify.app/)

---

## Stack

- **Vue 3** (Composition API)
- **Tailwind CSS**

---

## How to Run Locally

```bash
git clone https://github.com/eerikkodasma/speakly.git
cd speakly
npm install
npm run dev
```

## Tradeoffs and reasoning

Did do:

- Filtering / Search bar - I have done that before.
- Accessibility considerations - Can navigate with tab and activate filtering and buttons
- Mobile-responsive layout - Did it, but could be done better.
- Debounced search input - Important when you are dealing with big data.
- Score trends or historical rank tracking - Kind of showed some history with previous rank by arrows and colours.

Didn't do:

- TypeScript - Started doing project with vanilla javascript. Thought I would add TypeScript later, but ended up not doing it because limited time.
- UI animations - Have done any table UI animation. Didn't want to waste time on learning to do that in this task. Ended up just showing with arrows and colours.
- Export leaderboard to JSON - Have not done this.
- Virtualized rendering for large data sets - Didn't think it would be so important to do.

---

## Optional system design analysis (if the resulting product of this task was a full-stack app:)

### how would its API look like?

Only model you need is users model, that has all the FE user attributes as fields.

class LeaderboardUser(models.Model):

- user_name = models.CharField(max_length=100, unique=True)
- score = models.IntegerField(default=0)
- rank = models.IntegerField(default=0)
- previous_rank = models.IntegerField(default=0)
- points_gained = models.IntegerField(default=0)

### how would frontend receive updates from backend?

Querying with axios/TanStack Query/... .

1. Query initial leaderboard: Get http://speakly/api/leaderboard

2. Query every 10 seconds from the FE:

- 2.1 Get http://speakly/api/leaderboard/update-scores. BE gives some users more scores and return all the users back to FE.

- or

- 2.2. Post http://speakly/api/leaderboard/update-scores. Fe sends Users and scores they need to get. BE updates and just sends Users back

### how to make sure that the displayed rankings are updated consistently?

Always guery from the BE and don't Sort or order in the FE, that way the data is predictable.
