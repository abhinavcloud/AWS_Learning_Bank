# Primary LP - Deliver Result
# Secondary LP - 

# Story - Migration and Transformation of FlexCab Billing files to AWS Cloud 

### Situation:

>As part of Telstra's T30 modernisation roadmap, we needed to migrate FlexCab — a legacy mainframe billing system storing data in EBCDIC format — to AWS. The files were massive, up to 60-70GB each, containing thousands of billing accounts and thousands of charge line items per account, and needed to be transformed into JSON to feed the new digital bill presentment system.

### Task:

>My task was to build the pipeline to transport, parse, and convert this data reliably — with zero tolerance for billing errors reaching customers, given the scale and sensitivity of the data.

### Action:

>Our first plan was to use AWS Glue, since it fit the ETL pattern and would have been the faster path to build. Partway through evaluation, we found EBCDIC wasn't among Glue Crawler's supported formats — its built-in classifiers cover formats like JSON, CSV, and Avro, with nothing for mainframe binary encodings. Rather than force a workaround into a tool that wasn't built for this, I pivoted the architecture to EC2 instances, giving us full control over the parsing logic.

>Given the file sizes and volume, a mid-process failure was a real risk, and I didn't want to accept "restart from scratch" as the failure mode — that would risk delays and inconsistent state across billing accounts. I built a checkpointing layer using ElastiCache Redis that tracked which billing accounts had been successfully processed, the total count, and the next account to process. If a job failed partway through, we could resume exactly where it left off instead of reprocessing everything.

# Result:

>We delivered a system that processed 100% of billing accounts successfully. Post-migration billing disputes and complaints about incorrect bills stayed under 1%, and customer feedback on the new digital bill presentment improved by 35% compared to the legacy experience.