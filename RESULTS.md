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
| A1   | Smoke        | 1     | 0.48  | 370.15 ms | 421.63 ms | 0.00% | 100.00% (20/20) |
| A2   | Average load | 3     | 1.36  | 421.76 ms | 572.22 ms | 0.00% | 100.00% (332/332) |
| A3   | Stress       | 5     | 2.04  | 476.04 ms | 1.35 s | 0.00% | 100.00% (504/504) |
| A4   | Spike        | 5     | 1.04  | 407.90 ms | 1.50 s | 0.00% | 100.00% (196/196) |
| A5   | Soak         | 2     | 0.96  | 412.49 ms | 1.12 s | 0.00% | 100.00% (844/844) |

## Part B

1. The results table for A1 to A5 is in "Part A results" above. No test had a failed request or a failed check, and all thresholds passed.
2. In Step 0.2 one iteration is one request of about 0.3 s plus a 1 s sleep, so about 1.3 s. One user makes less than 1 request per second. With 5 users the rate was 3.70 requests per second (75 requests in 20 s), not 5.
3. Yes, a little. The p(95) grew from 421.76 ms (3 users, A2) to 476.04 ms (5 users, A3), and the max grew from 572 ms to 1.35 s. The request rate also grew, from 1.36 to 2.04 requests per second. There were still no errors, and both group thresholds passed. So 5 users is still inside the operational envelope of QuickPizza: the system slows down a little, but it does not fail. We cannot find the edge of the envelope, because we may not use more than 5 users.
4. The spike started at 30 s. In the 10 s window from 30 s to 40 s the average rose from about 335 ms to 439 ms and the p(95) rose from about 390 ms to about 903 ms (max 911 ms). In the next 10 s window the p(95) was already back to about 403 ms, and the average was 324 ms. So the response time returned to normal within about 10 to 20 seconds. Later, at 1 user, there was a separate slow window from 110 s to 120 s (max 1.5 s). It happened after the spike, so it looks like network or server noise and not a result of the spike.
5. No. The error rate stayed at 0.00% for the whole 11 minutes. The p(95) per 10 s window stayed between about 346 and 536 ms, with a few short spikes (for example 861 ms at 290 s and 921 ms at 420 s) that went back to normal at once. The first minutes (p(95) 375 to 392 ms after warm-up) and the last minutes (p(95) 354 to 386 ms) were about the same. A soak test of several hours could show a memory leak, a connection or file handle leak, a database or log file that fills up, or a slow growth of response time.
6. The average hides the slowest users. In this lab the averages were about 320 to 340 ms, but the max reached 1.5 s. The p(95) shows what the slow 1 request in 20 feels like, so a requirement such as "95% of requests within 1 second" is more honest and can be tested with a threshold.
7. First, a public site is shared with other users around the world, so its load changes all the time, while a lab system has no other traffic. Second, the internet adds network delay and jitter between my laptop and the server (about 270 ms minimum here), while a lab system on a local network has a small and stable delay. Hardware, caching and server size also differ.
8. A stress, spike or breakpoint test with hundreds of users can slow down or crash the site for its real users. It looks like a denial of service attack, it may break the site rules or the law, and it can cost the owner money. We need the owner's written permission, and only the owner can accept the risk.
9. Vague requirement: "The messaging app should send my message fast." Testable requirement: "With 5 users sending a message at the same time for 1 minute, 95% of send requests finish within 500 ms and 99% within 1 second, fewer than 1% of requests fail, and more than 99% of checks pass." K6 thresholds:
   ```
   export const options = {
     vus: 5,
     duration: '1m',
     thresholds: {
       http_req_duration: ['p(95)<500', 'p(99)<1000'],
       http_req_failed: ['rate<0.01'],
       checks: ['rate>0.99'],
     },
   };
   ```
