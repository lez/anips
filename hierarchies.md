# NIP: Hierarchies

`draft` `hierarchy` `curation` `clientside`

This NIP defines a standard for building social hierarchies and a way they can curate content collectively.

The aim is to come up with a scalable method for building large groups, for which hierarchy is a battle-tested solution. Thus, everything is public and plaintext: the structure, the content, the disputes and the decisions. Personal privacy is achieved via using pseudonyms.

Depending on the application, content curation can be just an abstract name for group moderation. These terms are used interchangedly in this NIP.

This is a client-based approach to build groups, no need for support by the relay. Groups can move between relays freely, they can be forked or combined together.

This NIP does not support adding score / confidence numbers to relationships, or any kind of ranking. It operates solely with boolean decisions, aka curations. Something or somebody is either in or out.

### Building the hierarchy

A hierarchy is created by a leader, who is the first member of that. Members can add other pubkeys to the level below them by issuing stamps that reference the pubkeys.

The result is a tree graph, which is rooted at the leader. Each member can show up a trail of valid stamps that leads from the leader to them.

There are usecases where this is all we need. For example when we want to control who can write to our relay.

The same hierarchy can be used in multiple applications.

### Content Curation

A hierarchy can produce some kind of curated content, just like a media company produces a newspaper.

The curated content is made up of what the hierarchy members post. Additionally, individual events can be approved / disproved using stamps that reference the event ids.

Clients are natural limits for the scope of the content curation. This means you can still post anything as kind 1 note, but in the reddit-like nostr app you should follow the guidelines set by the subreddit admins.

## Technical details

### Stamps

Pubkey stamp:
```js
{
  "kind": 77,  // stamp kind
  "tags": [
    ["p", "<stamped-pubkey>"],
    ["c", "<context>"],
    ["transitive", "false"],  // default: true
    ["revoked", "true"],  // default: false
    ["parent", "<pubkey>"[, "<pubkey>"...]],  // hints for pubkeys in levels above
    ...
  ],
  "pubkey": "<stamper-pubkey>",
  ...
}
```

Event stamp:
```js
{
  "kind": 77,
  "tags": [
    ["e", "<stamped-event-id>"],  // multiple
    ["c", "<context>"],
    ["revoked", "true"],  // default: false
	["parent", "<pubkey>"[, "<pubkey>"...]],  // hints...
    ...
  ],
  "content": "<reason>",
  "pubkey": "<stamper-pubkey>",
  ...
}
```

The context `"c"` describes what the stamp is valid for.
It is string that uniquely identifies the hierarchy. Either an event id, relay pubkey, or a group name is fine.

Pubkey stamps are transitive, unless the `["transitive", "false"]` tag is set. But in any case members can stamp individual events, which counts as valid by the hierarchy.

A stamp can be revoked by the stamper by adding the `["revoked", "true"]` tag.

Stamp events are transitive by default, which encourages taking on responsibilities. To maintain good quality of content, 1984 reporting/flagging of inappropriate behavior should be an integral part of keeping the conversation civil.

For cases where this process is too slow or the violation is obvious, shorter stamp trail beats the longer one. That means you can override stamps if you are higher up in the hierarchy. This opportunity should be used wisely and responsibly, only when needed, with clear reasoning.

For each stamper/stamped/context combination only the latest stamp event counts. Thus, it behaves as if it was a replaceable event with a `"d"` tag of `stamped/context`. But it's simpler this way.

If a member posts inappropriate content, it can be moderated out with a revoked event stamp. The revoked stamp must be originated from at least one level above the member. An event stamp is always stronger than a pubkey stamp that points to the same event and originated from the same level.

Additional machine readable metadata MAY be added to stamps as extra tags or any human readable text can be put into `content`.
### Relays
This NIP only specifies where stamps should be stored.
The client specific events are stored based on the client's own logic.

Any client specific events should be acompanied by the stamp trail that prove its place in the hierarchy. So the relay that returned the event should return the stamps, too, if REQested.

Stamps SHOULD also to be found on the read relays of the stamped pubkey or the author of a stamped event. (Or maybe just a notification?). This is to keep users in the flow of what's happening with their data. The users can take care of archiving / keeping the stamps alive.

Revoked stamps MUST be stored on the stamper's relays so that anyone can check if a stamp is revoked. However, all the stamps SHOULD be available on the stamper's write relays for listing.

## Querying

These are the requests a client has to do in order to find out if an event is in or out.

1. Fetch the client-specific events.
2. For each author, fetch the pubkey stamps, which were issued for them. For multi-level hierarchies repeat recursively until the leader is reached. Use `parents` as hints to initiate less number of queries.
3. Run a filter for event stamps for the events, which are still not approved. There MAY be a mechanism to select and show some of unapproved events on a different tab.
4. Check for revoked stamps for all the stamped pubkeys and displayed events. Look in stampers' relays.

Stamps MAY be saved to localstorage for a snappy UX.

The interpretation of these stamps MUST be identical in order to be able to use it in multiple usecases.

## Notes

When a transitive pubkey stamp is revoked, the whole sub-tree of curation might disappear. Clients should warn users of this before revoking the stamp and offer a way to re-stamp the pubkeys and content down the hierarchy.

Stamp events MAY be sent privately to users, relays or 
filtering algorithms using NIP-59 gift wraps.

Members MAY run algorithms to stamp content in or out. Bots are fine, too. They can improve onboarding experience for newcomers, etc.

## Forking

Hierarchies are forkable by default. If the leader gets compromised or disappears, another respected member can stand up as the leader and stamp the same pubkeys, which the previous leader stamped. This should be avoided if possible, because it divides the people. Better to settle arguments by disputes and rough consensus.

Alternatively there can be a pre-set policy (by the leader) about how to elect / who to point as new leader if the current one disappears. There will be a default policy as well, which the leader should take into account while building up the organization.
