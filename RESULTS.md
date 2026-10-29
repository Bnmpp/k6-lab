# K6 lab results

## Part 0

- Step 0.1, http_req_duration: avg=med=min=max=p(90)=p(95)=345.54 ms (one request)
- Step 0.2, requests, rate and p(95), and why the rate stays below 5: 75 requests, 3.70 requests per second, p(95) 337.96 ms. One iteration is one request (about 0.3 s) plus a 1 s sleep, so one user needs about 1.3 s per iteration. That is less than 1 request per second per user, so 5 users give about 3.7 requests per second, not 5.
- Step 0.6, exit code of the normal run and of the run with P95=300: normal run exit code 0 (p(95) 422.51 ms, under 1500 ms). Run with P95=300 exit code 99 (p(95) 421.1 ms, over 300 ms, so the threshold failed).
- Step 0.7, rules that passed, p(95), p(99), exit code after your change:
- Step 0.8, what was wrong, and how the checks showed it:

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
