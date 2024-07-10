+++
title = 'PM2'
date = 2024-07-09T00:20:22+09:00
draft = true
+++

## Cron Job 등록하기

🔺 pm2 run.json 
- 이 경우 dev에만 넣어주면 됨. (dev -> main 요청)
- `pm2 start run.json` 으로 등록하면 알아서 크론탭으로 실행 후 종료함.
- pm2 특정 앱만 로그 보는법: pm2 list -> `pm2 logs 14`

createApplicationContext 별거 없음.
배치를 실행할 유스케이스 만들고
.get으로 가져와서 실행하면 끝.


```json
{
    "apps": [
		{
			"name"        : "sync-date-batch",
			"script"      : "./dist/bin/dev-sync-date.js",
			"cwd"         : "/home/ubuntu/ncs-rank-query-server/",
			"cron_restart": "*/5 * * * *",
			"watch"       : false,
			"autorestart" : false,
			"env": {
				"NODE_ENV" : "development"
			},
			"instances": 1,
			"merge_logs" : true
        }
    ]
}
```