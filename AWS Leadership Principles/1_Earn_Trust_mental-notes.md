Here's one flowing paragraph you can say end-to-end, with every metric baked in:

"I owned the Pricing and Discount Engine for a 10M-subscriber telco, built on Lambda for cost and simplicity — normal traffic ran around 12 requests per second. 


During Eid, marketing's promo notifications drove sharp spikes: subscribers opening the app within minutes, each session firing multiple pricing calls, pushing us to about 5,000 requests per second for 10 to 15 minutes at a time, three to four times a day. 


We hit 15% throttling and 300 to 500 millisecond cold starts. 

I owned it immediately — stakeholders informed within 30 minutes, rolled back to the legacy stack to stop revenue loss. 

The real mistake was mine: I'd launched on Lambda's default on-demand scaling and never configured it for a peak we already knew was coming. 

So I sized the actual fix from first principles — with a cache in front of the catalog data, a warm lookup ran about 80 milliseconds, so 5,000 RPS times 0.08 seconds is roughly 400 concurrent executions needed at peak. I provisioned 500 with a buffer, set reserved concurrency to 600 so the function couldn't be starved by others, and scheduled it to scale up from a baseline of 50 about 10 minutes ahead of each known promo window and back down after — all shipped in one two-week sprint. 


Result: throttling dropped to zero, availability held at 99.9% through every later peak, and we avoided the estimated $200k a day in revenue risk. I turned it into a launch checklist so known-event traffic patterns get load-tested and scheduled-scaled before anything ships again — owning that planning gap and being transparent throughout is what rebuilt trust."