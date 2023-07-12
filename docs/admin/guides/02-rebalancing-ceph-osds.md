# Rebalancing Ceph OSDs
From time to time, Ceph may get out of balance. This will look like various pools warning that they are full or near-full, and in particular OSDs reporting a full or near-full status.

This is meant to be taken care of largely automatically by Ceph's `balancer` module for the ceph `mgr`, however in practice one may still need to intervene from time to time (as we had to a week or so before this guide was written, in late June 2023).

The key command to know is this:

`ceph osd reweight-by-utilization 120 0.4 12 --no-increasing`

and a slightly more drastic version:

`ceph osd reweight-by-utilization 120 0.4 12`

In the command `ceph osd reweight-by-utilization 120 0.4 12 --no-increasing`:

```markdown
120: This is the threshold of over-utilization percentage. If an OSD's usage is above 120% of the average utilization, the command will try to decrease its weight, therefore less data will be placed on it. The percentage is calculated based on the total usage of all OSDs in the system.

0.4: This is the maximum change that can be made to an OSD's weight in a single invocation of the command. Here, no single OSD's weight will be changed by more than 40%.

12: This is the max number of OSDs that can be changed in one command execution. The command will adjust the weights of up to 12 OSDs in a single run.

--no-increasing: This option indicates that the command should only decrease the weight of OSDs, not increase. This is useful when you want to ensure that no OSD gets more data than it currently has, which might be important when some OSDs are close to their storage limit.
```

So, in summary, the command will reduce the weights of up to 12 most utilized OSDs (those with utilization over 120% of the average) in the cluster by a maximum of 40%, without increasing the weight of any OSDs. The weight reduction makes the data placement algorithm favor other OSDs for storing new data, which in turn should lower the utilization of the over-utilized OSDs over time.
