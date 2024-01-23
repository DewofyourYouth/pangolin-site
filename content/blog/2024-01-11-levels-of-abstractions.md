---
title: Levels of Abstractions
date: 2024-01-12T12:10:42.900Z
description: A quick illustration of a code smell.
draft: false
showHero: true
showComments: true
thumbnail: /img/maps.jpeg
tags:
  - news
---
In my last post on development guidelines, I referred to the idea that functions should maintain a singular level of abstraction. To clarify what I mean by this, I've decided to provide a contrived example for illustration purposes.

- - -

When you have a function to do something - you should write it in a manner where you're maintaining a similar level of abstraction - shifts in the level of abstraction make you code less clear to the reader and cause more bugs.

If you're doing something high level, like a data transformation or handling a user request - and then you suddenly have to drill down into the implementation details of a tcp request - or deal with chunking binaries - this causes a mental toll that makes solving problems on the higher level more difficult. 

And if you're on the lower level, and then you jump into a higher level abstraction, this also imposes a mental toll. You're just trying to correctly manage some bytes and now suddenly you have to worry about what's going on in user space! WTF!?

Example:

Here is a function

```python
def handle_logs(logs_dir: str) -> dict:
	data = {}
	engine = sqlalchemy.create_engine(
	f"postgresql+pyscog2://${os.getenv("DB_HOST")}@{os.getenv("DB_USER")})..."
	)
		with engine.connect() as conn:
		    for file_name in os.listdir(logs_dir):
		        with open(f"f{logs_dir}/{file_name}") as json_file:
			        logs = json.loads(json_file)
			        for log in logs:
				        if log["level"] == "INFO" or log["level"] == "DEBUG":
					        if environment == "DEV" or environment == "STAGING":
						        q = conn.execute(
						        """
						        INSERT INTO LOGS (level, message, timestamp)
						        VALUES (?, ?, ?)""" % (
						        log["level"],
						        log["message"],
						        date.strptime(log["timestamp"], time_format)))
						if log["level"] in [
							"WARN",
							"CRITICAL",
							"ERROR",
							"FATAL"
							]:
							requests.post(
							f"https://{company}.slack.com/\
							{notifications_endpoint}",
								 {"X-Token": my_token},
								 {"message": log, "channel": "dev-team"},
							 )
								q = conn.execute("""
								INSERT INTO LOGS (level, message, timestamp)
								VALUES (?, ?, ?)""" % (
								log["level"],
								log["message"],
								date.strptime(log["timestamp"], time_format))
								)
					data[q.primary_key] = f'{log["type"]}, \
							{log["timestamp"]}, {log["message"]}'
	return data
```

This function is aptly named `handle_logs` and it handles logs. It then returns a dictionary of data - where the key is the log insertion's primary key in the database.

But *how* does it handle the logs?
What does it do with the logs?

Since this example is contrived, and there isn't any external application logic to consider, so you probably could figure all that out relatively easily. But it's still (at least a tiny a bit of) a mental lift. If you were focused on **fixing a problem**, rather than just understanding the everything about the function, you could easily misunderstand, or just miss something. For example, you could potentially miss the fact that, for certain types of logs, the function makes a post to a Slack API. If there was a problem with the Slack notifications - you might not even think to look at this function in the first place, in the context of a broader app.

Now consider the following code:

```python
def handle_logs(logs_dir: str) -> dict:
	data = {}
	logs = parse_logs_from_logs_directory(logs_dir)
	for log in logs:
		if is_log_scary(log):
			notify_team_on_slack(log, channel="dev-team")
		primary_key, log_string = insert_log_into_db(log)
		data[primary_key] = log_string
	return data
```

We haven't actually changed any of the code the implementation. All of the implementation details are still spelled out - but they're in functions that accept the logs as input. The developer, without thinking too hard, can quickly see that this function:

1. Parses all the log files from a logs dictionary.
2. If the log type is "scary" - it notifies the "dev-team" channel on Slack.
3. It inserts the log into a database.
4. It adds a record of this into a dictionary called `data`
5. It returns the `data` dictionary

If you needed to then see how it does any of such things, you could inspect that specific function. But more importantly, now that you mental bandwidth is free from figuring out what it's doing, you could focus on the issues that are actually more appropriate to this problem space like:

1. What happens to the log file after it's parsed? Does it just stay there forever?
2. What happens if the Slack server is down? Does the function throw an exception and quit?
3. What insures logs aren't parsed, entered, or Slack notified twice?
4. Whatever other reason you're looking at this function.