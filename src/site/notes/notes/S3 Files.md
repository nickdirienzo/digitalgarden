---
{"dg-publish":true,"permalink":"/notes/s3-files/","created":"2026-04-07T14:51:34.513-07:00","updated":"2026-04-07T15:02:02.858-07:00"}
---

With AI agents depending on filesystems for searching, editing, and storing context, it was only a matter of time before AWS had a cloud-native answer.

Today is that day: AWS launched S3 Files ([All Things Distributed](https://www.allthingsdistributed.com/2026/04/s3-files-and-the-changing-face-of-s3.html), [AWS](https://aws.amazon.com/s3/features/files/)). 

This is a big deal. Agents work on filesystems. They `ls`, `grep`, `cat`. We've watched Claude Code and Codex shift the industry from semantic search back to grep, which spawned a wave of tools built as CLIs instead of MCP servers. As an industry we've learned that agents are most capable when they operate like developers do, on files.

S3 Files maps buckets and prefixes to NFS mounts. That means you can create filesystem sandboxes backed by S3 durability and attach them to Lambda, ECS, whatever. Lambda becomes stateless execution with a durable filesystem. That's a meaningful architectural primitive.

For teams building agentic infrastructure on AWS, the implications are immediate. Personalized agent contexts, scratchpads, durable working directories; all backed by S3 durability and scoped per-tenant with existing IAM and encryption primitives.

The cloud is catching up to how agents actually work.
