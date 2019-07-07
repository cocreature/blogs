# DAML choice annotations : Introducing pre- and post-consuming choices

Choices in DAML can be "consuming" or "non-consuming" with respect to the contracts on which they are exercised. In short, a consuming choice on a contract leaves the contract inactive, a non-consuming choice does not. By default, a choice is assumed to be consuming. To indicate a non-consuming choice, DAML provides the `nonconsuming` keyword. Recently, DAML choice annotation syntax has been added to further categorize choices as "pre-consuming" or "post-consuming". This post reviews the consuming concept and explains the meaning of the newly added `preconsuming` and `postconsuming` keywords. The example code used here can be found in this [Gist](https://gist.github.com/shayne-fletcher-da/fe321313ba085693d19afee4ecdf5ea0)

*[Note : The code here uses the new "flexible controller" syntax first introduced in [this blog post](https://digitalasset.atlassian.net/wiki/spaces/DEL/blog/2019/03/28/822509571/DAML+Does+Yoga+An+Introduction+to+Flexible+Controllers?focusedCommentId=846987557#comment-846987557)]*.

## Consuming choices

Normally, exercising a choice on a contract amounts to actions that terminate the contract or replace the contract with one or more new contracts. For this reason, by default, exercising a choice on a contract is assumed to "consume" the contract. For example, suppose a contract representing some kind of offer to purchase goods or services with provision for the provider to withdraw the offer.
```haskell
template Offer
  with
    supplier : Party
    customer : Party
  where
    signatory supplier

    choice Withdraw : ()
      controller supplier
      do
        return ()

    ...
```
DAML guarantees that any `Offer` instance on which a `Withdraw` choice has been exercised is thereafter "inactive" meaning any attempts to exercise further choices on that instance must fail. Exercising a `Withdraw` is said to "archive" the contract.

## Non-consuming choices

It's sometimes the case though that exercising a choice on a contract instance does not imply the instance should be archived. For such a choice, we can indicate that with a `nonconsuming` annotation. To illustrate, here's a simple model of a leasing agreement between a landlord and tenant. The landlord may like to extend to the tenant the opportunity to "refer a friend" in order to obtain a cash-premium (say) so we write that into the contract like so.
```haskell
template Lease
  with
    tenant  : Party
    realtor : Party
    guarantor : Party
  where
    signatory realtor, tenant
    observer guarantor

    nonconsuming choice ReferAFriend : ContractId ReferAFriendOffer
      controller realtor
      do
        create ReferAFriendOffer with leaseCid = self, ..

    ...

template ReferAFriendOffer
  with
    realtor : Party
    tenant  : Party
    leaseCid : ContractId Lease
  where
    choice Accept : ...
      ...

   ...
```
In this case, no matter how many times a `ReferAFriend` choice is exercised on a `Lease` contract instance, that lease contract remains active.

## Introducing pre- and post-consuming choices

Suppose we wish to write a clause into our `Lease` agreement allowing for its termination. Put yourself in the role of the developer contracted by the realtor to model their business process that mandates that on termination not only is the agreement archived but also a `ReferAFriend` offer is issued. That would lead us to try to write something like the following.
```haskell
template Lease
    ...

    choice Terminate : ContractId ReferAFriend
      controller realtor
      do
        exercise self ReferAFriendOffer -- This won't work!
```
As indicated by the comment though, what we wrote won't cut it. The reason is, at the point at which we try to exercise the `ReferAFriendOffer` on the lease contract, the lease has already been archived! The semantics of this auto-archiving are "pre-consuming". The contract on which the choice is being exercised is archived before the choice body is executed and so the attempt to exercise the `ReferAFriendOffer` on the lease in the choice body fails because the lease is inactive.

In fact, the newly introduced `preconsuming` choice annotation is provided for those that wish to be explicit in their definitions. That is,
```haskell
    choice Terminate : ContractId ReferAFriend
      controller realtor
      do
        exercise self ReferAFriendOffer
```
and,
```haskell
    preconsuming choice Terminate : ContractId ReferAFriend
      controller realtor
      do
        exercise self ReferAFriendOffer
```
mean the same thing and as we have seen, the semantics of this archival scheme hinder our ability to naturally express the business process.

What's lacking here is a choice archival scheme with slightly different semantics -- one that auto-archives its contract but not before the choice body has been executed. This is provided for by the newly added `postconsuming` annotation. Now we have the tools to express our intent.
```haskell
    postconsuming choice Terminate : ContractId ReferAFriend
      controller realtor
      do
        exercise self ReferAFriendOffer -- Ok!
```
What we see here is that the `postconsuming` annotation, in contrast to a `preconsuming` annotation, means that the contract remains active for the duration of the choice body and is archived thereafter.

There is one subtlety left to explain regarding privacy rules with respect to post-consuming choices. Like non-consuming choices, contracts created in the choice body are only known only to the signatories and controllers of the contracts and not made known to the observers of the contract on which the choice was exercised. That means, in terms of the example, a guarantor will not be privy to the creation of a `ReferAFriendOffer`  raised by exercising `Terminate` on a lease but, as a stakeholder, will be privy to the archiving of the `Lease` itself.

## A remark for ledger writers

In practice, DAML-LF only has preconsuming choices. Post-consuming choices are implemented by code generation during the DAML to DAML-LF desugaring process. Don't worry if this remark doesn't mean anything to you as a DAML developer - you can safely ignore it!

### Summary

In this note we have revisited the notions of "consuming" and "non-consuming" choices. Consuming choices render their contracts inactive, non-consuming choices have no affect on their contracts. Consuming choices can be further nuanced into "pre-consuming" and "post-consuming" choices. Pre-consuming choices archive their contracts before their bodies are executed and post-consuming choices archive their contracts after their bodies are exectued. Annotations `preconsuming`, `nonconsuming` and `postconsuming` can be attached to `choice` definitions to select between the different possibilities. If you don't annotate a choice then it defaults to post-consuming.
