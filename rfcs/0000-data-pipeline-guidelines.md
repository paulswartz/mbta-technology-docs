- Feature Name: `data-pipeline-guidelines`
- Start Date: 2025-01-16
- RFC PR: [mbta/technology-docs#0000](https://github.com/mbta/technology-docs/pull/0000)
- Asana task: [asana link](https://app.asana.com/)
- Status: Proposed

# Summary
[summary]: #summary

Application teams are responsible for providing generic events, supporting any potential consumers. LAMP is responsible for the data pipeline, including processing application data into tabular formats. If the application team needs to include their data in LAMP and the LAMP team does not have capacity, the application team should use the Away Team model to build the middleware for processing their data into LAMP.

# Motivation
[motivation]: #motivation

This RFC clarifies the pattern for applications to provide data to be used in analysis by TID and other teams at MBTA. It also proposes a model for application teams to support the data analysis pipeline without unduly burdening the LAMP team.

# Proposal
[proposal]: #proposal

TID currently uses two general integration patterns for sharing real-time data between applications: GTFS-RT feeds (uploaded to S3 or MQ) and CloudEvents (shared via Kinesis). CloudEvents and Kinesis were standardized as a part of [RFC19](https://github.com/mbta/technology-docs/blob/main/rfcs/accepted/0019-event-driven-architecture.md) (and adopted earlier in [RFC4](https://github.com/mbta/technology-docs/blob/main/rfcs/accepted/0004-socket-proxy-ocs-cloudevents.md%5D), [RFC5](https://github.com/mbta/technology-docs/blob/main/rfcs/accepted/0005-kinesis-proxy-json.md), and [RFC18](https://github.com/mbta/technology-docs/pull/18)).

As a part of that RFC, it was outlined that:
- events should be generic, decoupling any consumers from the producer
- changes to events need to be backwards compatible
- consumers need to maintain their own internal state based on the events

This does create some burdens on consumers, and this burden impacts the LAMP team more strongly as they are a consumer of events from multiple teams. In order to support the analytic needs of the department, we need a model which supports both.

The expectation is still that the code which processes events for analysis will live in LAMP. This includes both light-touch processing (flattening JSON into columns) and higher-touch processing (restructuring the data to better support downstream analysis). Likely both types are important: an archive of the raw events and a processed version more amenable to analysis based on stakeholder needs. 

One option would be a consumer of the events (such as a Lambda) which restructures the event into a slightly different (but still JSON) format and writes it to a new stream. This would allow the existing LAMP JSON-flattening code to continue to work as it currently does.

Given that the LAMP team has their own roadmap and limited capacity, application teams should be prepared to form an [Away Team](https://pedrodelgallego.github.io/blog/amazon/operating-model/away-team-model/) to support LAMP in consuming their events if that needs to happen on a particular timeframe. 

# Drawbacks
[drawbacks]: #drawbacks

As of the writing of this RFC, there are no engineers on the LAMP team. This proposal would require all work for data analysis to be performed by Away Teams until that changes.
# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

There are two competing desires for the integration between the application and LAMP:
- the application wants to provide generic events, such that they can provide a single event stream
- LAMP prefers data which is already suitable for analysis (easily converted to a flat table)

Teams could provide multiple events: some for consumption by non-analytic applications (other application teams, external consumers), others for consumption by LAMP. This results in a duplication of effort across all teams: each application team providing data to LAMP needs to have multiple data streams. The approach laid out here allows LAMP to share the processing of events across multiple applications if needed, limiting the duplication of work required.
# Prior art
[prior-art]: #prior-art

The MBTA already has some of this type of processing happening already in MBTA360. Especially for vendor applications, it's not always possible to adjust their data formats. Instead, the teams putting the data into MBTA360 are responsible for structuring it such that it can be analyzed by others.

# Unresolved questions
[unresolved-questions]: #unresolved-questions

None at this time.

# Future possibilities
[future-possibilities]: #future-possibilities

In the future, we may have more standardized analysis patterns or steps in the pipeline. For example, there may be steps in the pipeline which pull in additional data to enhance the incoming events before writing the data to S3 or other sources.
