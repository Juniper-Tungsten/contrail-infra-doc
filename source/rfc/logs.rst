Logs
====

Problem
-------

The log server is full way too often. We're currently running on a 14-day policy, even though we should be keeping at least 30 days of logs.

Current setup
~~~~~~~~~~~~~

We run on top of our Hetzner hypervisor with:

- 4 cores
- 8GB of RAM
- 400GB of disk (LVM)

Notes:

- there's only 600GB free on the hypervisor
- 2nd hypervisor has 800GB free
- we seem to be generating from 20 to 40GB of logs per day (as of 2018-05-04)

Retention policy
----------------

We should keep 30 days of logs for reviews - that should be enough to be able to merge a review properly.

Nightlies
~~~~~~~~~

Nightly logs should follow the very same policy that build artifacts (packages and containers) will follow to ensure that we have all information we need historically. What's more, we should consider keeping nightly logs for a longer period of time (60 or 90 days), as someone might use an older build (even though we don't provide any support for them).

Quick fix
---------

We should increase server size to 800GB (allowing some wiggle room for other machines to expand if necessary) which should last us during quiet time (20GB/day gives us 600GB with 30 days). In the meantime we should focus on a proper fix.


Proper fix
----------

Storage
~~~~~~~

Because it's extremely difficult to properly estimate the log growth due to variety of possible reasons (amount of jobs, reviews, flakiness), we should move to an object storage solution as soon as possible. 

One possibility would be AWS with S3. Having object storage mounted as fs (s3fs) allows us to store the logs same as before, but with no limits other than our own enforced policies as the storage is pretty much unlimited.

Log curator
~~~~~~~~~~~

Log curator's reponsibility is to plainly remove the logs after a defined period of time. To be able to maintain two different policies for nightly and reviews at the same time, we have two options:

- separate review logs (e.g. /var/www/logs) and nightly logs (e.g. /var/www/nightly-logs)
- expand log curator to be able to enforce different time policies for /var/www/logs and /var/www/logs/periodic-nightly

If we are to use S3, we can use `S3 Object Expiration <https://aws.amazon.com/blogs/aws/amazon-s3-object-expiration/>`_ instead of Log Curator, allowing us to drop it completely.
