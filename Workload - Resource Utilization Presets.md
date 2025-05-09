## General QoS class remarks

In order to ensure optimal workload behavior, the workload should utilize at least the `Burstable` QoS class.

## CPU Limits

**Source:** https://tinyurl.com/k8s-cpu

![[k8s-cpu-limits-vs-requests.png]]

## Three colorful analogies about Kubernetes CPU Limits

Marcus and Teresa are travelling in the desert. They have a magical water bottle that produces 3 liters a day. Each person needs 1 liter a day to survive.

**Story 1 - without limits, without requests:** Marcus is greedy so he drinks all the water before Teresa can drink any. Teresa dies of thirst. This is because there were no limits or requests. Marcus was able to drink all the water,  cause CPU starvation, and Teresa was CPU throttled.

**Story 2 - with limits, with or without requests:** Teresa gets very ill one day and needs some extra water. Marcus drinks his one liter and there are two liters left over. Teresa drinks one liter and now there is one liter remaining. Marcus wont let Teresa drink it because her limit is 1 liter per day so she dies of thirst. This is what happens when you have CPU limits. Resources are available but you aren't allowed to use them.

**Story 3 - without limits, with requests:** Marcus gets very ill and needs extra water one day. He tries to drink the entire bottle but is stopped when only 1 liter remains in the bottle. This is saved for Teresa because she needs 1 liter a day. She drinks her 1 liter. Nothing remains. They both live. This is what happens when you have no CPU limits but you do have requests. All is good.

**The above stories are surprisingly precise analogies for why CPU limits are considered harmful.**

Don't like analogies? I wrote a more technical explanation of this when explaining the Prometheus CPU throttling alert, [CPUThrottlingHigh](https://github.com/robusta-dev/alert-explanations/wiki/CPUThrottlingHigh-(Prometheus-Alert)).