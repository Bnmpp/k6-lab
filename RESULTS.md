# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=med=min=max=p(90)=p(95)=345.54 ms (one request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.70 requests per second, p(95) 337.96 ms. One iteration is one request (about 0.3 s) plus a 1 s sleep, so one user needs about 1.3 s per iteration. That is less than 1 request per second per user, so 5 users give about 3.7 requests per second, not 5.
- Step 0.6, exit code of the normal run and of the run with P95=300: normal run exit code 0 (p(95) 422.51 ms, under 1500 ms). Run with P95=300 exit code 99 (p(95) 421.1 ms, over 300 ms, so the threshold failed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: all 3 rules passed (failure rate under 1%, p(95)<1000 with p(95) = 431.21 ms, p(99)<1500 with p(99) = 618.78 ms), exit code 0. I changed p(95)<1000 to p(95)<10 in a temporary copy. The p(95) rule failed (p(95) = 441.88 ms) and the exit code was 99.
- Step 0.8, what was wrong, and how the checks showed it: The script sent the lowercase names 'pepperoni' and 'knife', but the API knows 'Pepperoni' and 'Knife' (explore.js showed this). The API did not match the lowercase names, so it ignored the exclusions. Before the fix, 35 of 225 checks failed (84.44% passed): 'no pepperoni' failed 11 times and 'no knife needed' failed 24 times. After I changed the request to 'Pepperoni' and 'Knife', all 216 checks passed. The API picks a random pizza, so one manual request can give a good pizza by luck (a bad pizza appeared in only about 15% to 32% of requests). Under load there are many requests, so the rare wrong answers show up in the failure counts.

## Part A results

| Step | Test type    | Users | Req/s | p(95) | Max | Failed | Checks |
| ---- | ------------ | ----- | ----- | ----- | --- | ------ | ------ |
| A1   | Smoke        | 1     |       |       |     |        |        |
| A2   | Average load | 3     |       |       |     |        |        |
| A3   | Stress       | 5     |       |       |     |        |        |
| A4   | Spike        | 5     |       |       |     |        |        |
| A5   | Soak         | 2     |       |       |     |        |        |

## Part B

1.
2.
3.
4.
5.
6.
7.
8.
9.
