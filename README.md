# Rulealize.RuleSet.SignedRequest

A request that may not be granted until enough people have signed it — a
[Rulealize](https://github.com/reny-develop/Rulealize) rule set **made of two others**.

| | |
| --- | --- |
| Rule set id | `Rulealize.RuleSet.SignedRequest` |
| Package | [`Rulealize.RuleSet.SignedRequest`](https://www.nuget.org/packages/Rulealize.RuleSet.SignedRequest) |
| Inputs | none of its own |
| Holds | [`Request`](https://www.nuget.org/packages/Rulealize.RuleSet.Request) as `req`, [`Signatures`](https://www.nuget.org/packages/Rulealize.RuleSet.Signatures) as `sigs` |
| Draws on | State, Comparison, Record, Logic |

```json
"uses": [
  { "ruleSet": "Rulealize.RuleSet.SignedRequest", "version": "^1.0", "as": "approval" }
]
```

`rulealize restore` fetches this and both of the documents it holds, by identifier, from the
feed. **`as` is not optional** — an alias defaults to the identifier and may not contain a `.`.

## It is the guard, and nothing else

No state of its own and no inputs of its own. Everything it offers is one of its components',
under the name it calls them by:

```
req.raise   req.grant   req.deny   sigs.sign   sigs.decline
```

What this document adds is **when** each is offered.

**Nothing is granted unsigned.** `req.grant` waits for the signatures to be in.

**Nothing is signed before there is something to sign.** `sigs.sign` and `sigs.decline` wait
for a request to be under review. Without that, a case could collect its signatures first and
then raise the request they had already approved — the shape of every rubber stamp there has
ever been.

Neither document could say either. The request half has never heard of a signatory; the
signing half has no idea what is being signed. Both are correct on their own and the process
they describe together is not, which is what composition is for.

## Walking it

```
draft            req.raise(subject: budget-q3)
                 req.raise(subject: budget-q4)

under review     req.deny
                 sigs.sign(who: ann)   sigs.decline(who: ann)
                 sigs.sign(who: bo)    sigs.decline(who: bo)
                 sigs.sign(who: cy)    sigs.decline(who: cy)
                 ← no req.grant: nobody has signed

two of three     req.grant             ← now it appears
                 req.deny

granted          terminal (granted)
```

## What it deliberately does not do

**A refused signature does not deny the request.** It leaves `req.deny` as the only move, so
somebody has to actually deny it. The refusal is a fact about the signatures; the denial is a
decision about the request. Collapsing them would lose the difference between *nobody would
sign* and *we decided against it*.

**Nothing is driven with `fires`.** Signing and granting are two decisions, made by two people
at two times. A composite that made them one transition would be modelling a shorter process
than the one being run — which is what `fires` is for, and this is not that case.

## The instance

Both components' state is a field of this one's, so a case is still one document:

```json
{
  "$schema": "rulealize/state/v1",
  "ruleSet": "Rulealize.RuleSet.SignedRequest@1.0.0",

  "data": {
    "req":  { "data": { "subjects": ["budget-q3", "budget-q4"], "stage": "draft", "subject": null } },
    "sigs": { "data": { "signatories": ["ann", "bo", "cy"], "required": 2, "signed": [], "outcome": "open" } }
  }
}
```

Who may sign, how many it takes and what may be requested are all here and none of them are in
any of the three rule sets. Two signatures or five, three approvers or thirty, budgets or
deployments — same documents, different state.

## Where it fits

| The request is for | signed by |
| --- | --- |
| a payment above a threshold | two officers |
| a production deployment | a reviewer and someone on call |
| access to a system | the owner and a manager |
| a refund | anyone senior enough, twice |

And it can be held in turn: a roster that fills a shift only where a signed request asked for
it holds this the way this holds its own two.

## Trying it

Needs [Rulealize.Cli](https://github.com/reny-develop/Rulealize.Cli) 0.8.0 or later, which is
the version that fetches what a `uses` names:

```sh
dotnet tool install -g Rulealize.Cli
rulealize restore src/Rulealize.RuleSet.SignedRequest/ruleset/signed-request.json
rulealize play    src/Rulealize.RuleSet.SignedRequest/ruleset/signed-request.json --state state/example.json
```

The restore fetches `Rulealize.RuleSet.Request` and `Rulealize.RuleSet.Signatures` from
nuget.org and writes them beside the document, because that is where it resolves them from.

## Building the package

```sh
dotnet pack src/Rulealize.RuleSet.SignedRequest -c Release
```

A package with no `lib` folder holding this one document, and **no reference to the two it
holds**. `uses` names them by the identifier they are published under, so a restore reaches
the feed without a project file saying so — a `PackageReference` would be a second place for
that to be written down and a second place to get it wrong.

## License

Apache-2.0, and it covers the document as much as everything else here. The two rule sets it
holds are licensed by whoever published them, which in this case is the same person.
