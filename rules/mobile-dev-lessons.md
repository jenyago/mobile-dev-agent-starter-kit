# Mobile Dev Lessons (iOS/macOS App Store shipping)

About 20 distilled, generic, reusable lessons — each stated as a rule with its reasoning, no incident story attached. Verified against real shipping experience; treat these as the floor, not the ceiling, of what you'll learn shipping your own apps.

## Build & code

1. **Always target the Simulator, with a real current device name, for automated builds.** A physical device requires provisioning and signing that an automated session usually can't complete unattended — simulator builds are the reliable default for anything scripted.
2. **Treat a generated Xcode project file as a build artifact, not a source file.** If you use a config-driven project generator, never hand-edit the generated project — edit the source config and regenerate. This also means: regenerate after adding any new source or resource file, before rebuilding. A file that exists on disk but isn't declared in the generated project doesn't get bundled, and the failure mode is a confusing "missing file" error rather than an obvious one.
3. **Run a clean build (not just a build) after any bulk edit (5+ files).** Incremental compiler caches show stale diagnostics after bulk changes — a clean build is the only trustworthy result.
4. **Pre-archive hygiene: grep for placeholder/test-only strings, and clean derived build data before archiving multiple apps back-to-back.** A placeholder value shipped in a configure() call crashes for the reviewer, not for you — your own dev environment already has the real value cached elsewhere so you won't see it. Separately, stale intermediate build products from one app's build can leak into the next app's build in the same session if you don't clean between them.

## Paywall & purchase flow

5. **A bare paywall view that fetches its own offering internally can hang indefinitely.** Always pass the offering explicitly, and add a timeout with a visible fallback state — this exact bug class has caused real "app didn't load" rejections that looked fine in a quick manual test with good network conditions.
6. **Transaction lifecycle, both ends matter:** start the transaction-update listener at app launch (not just when the paywall is shown, or a purchase/restore completed elsewhere is missed), and call the transaction-finish step for every verified transaction (an unfinished one re-delivers on every future app launch, which reads as a bug even though the purchase itself succeeded).
7. **Never show the paywall on the first launch.** Show real value first; gate the paywall behind a later session. Paywall-on-launch measurably hurts retention and conversion both.
8. **Subscription terms must be visible before the purchase button, not just linked somewhere else.** This is a platform requirement, not a style choice.
9. **Configuration existing is not proof of a working paywall.** Three separate things each have to work independently: an entitlement with real products attached, a *published* paywall template (a configured product catalog does not automatically publish one — they're separate steps), and a real sandbox purchase that survives an app relaunch. Verify each explicitly; none of them implies the others.

## App Store Connect mechanics

10. **A 409/403/404 from the App Store Connect API on a known portal-only operation means stop, not retry.** Some operations (creating a new app, certain compliance declarations, some rejected-state edits) are portal-only regardless of API credentials or account role.
11. **Orphaned submissions: check before every new submission, and never retry a failed submit-for-review call.** There's a hard cap on concurrent review submissions per app, shared across every platform combined, not per-platform — and each failed retry can silently create a fresh orphan that consumes one of those slots, often not cleanable except through the portal.
12. **Nothing carries over between builds or versions.** Every new version needs a genuinely fresh, uniquely-numbered build — even a metadata-only fix with zero code changes, since a build already attached to a released version can't be reattached to a new one. Export compliance is the same story: it must be re-set on every single new build, never assumed to inherit from the last one.
13. **Locale updates are atomic, and release-notes text doesn't travel with them.** Always patch every locale in the same pass — a partial update commonly forces a second correction cycle to catch up the rest. Separately, "what's new" text does not automatically carry over when a new version is created from a prior one the way description/keywords usually do — set it explicitly for every locale even on a release with no user-visible changes (a generic "bug fixes and improvements" is fine for minor-market locales).
14. **A macOS app must be submitted under the macOS platform identifier, never the iOS one.** Submitting under the wrong platform creates an orphaned version that can't be deleted through the API.
15. **A tablet-size screenshot slot can silently accept a phone-framed image without erroring.** Visually verify every screenshot landed in the correct device slot after upload — an accepted-but-wrong-size upload is a rejection risk, not an upload error you'd otherwise notice.
16. **A beta/pre-release OS toolchain can archive and upload a build successfully, and only get refused at the actual review-submission step.** Check you're on a fully-released, non-beta toolchain before archiving for App Store submission — otherwise the entire archive-upload-attach pipeline is wasted work.
17. **In-app purchase must ride the same submission as the app version it belongs to.** A "we cannot locate the in-app purchase" style rejection almost always means the subscription/IAP's own configuration state isn't submittable yet — check its state directly rather than assuming the code or the build is at fault.

## Store listing & content

18. **Research keywords natively in the target language — never translate from another language.** The store's search matching is language-specific; a keyword that works in one language is often invisible in a different-language storefront's search, even with an otherwise fully localized listing.
19. **Never mention a third-party brand name in your subtitle, and count its characters yourself before pushing.** Brand names are a documented rejection trigger even to imply compatibility; separately, the subtitle field can silently truncate an over-length value rather than rejecting the update, so a broken-looking listing can go live with no error at all.
20. **A full-screen error overlay reads to a reviewer as "the app didn't load."** Use an inline error card with auto-retry instead — this is one of the most common, and most avoidable, app-completeness rejections.

## Rejection communication

21. **Rejection reasons live in your email/Resolution Center, not in any API field.** The API exposes review state, never the reviewer's actual message text — plan to retrieve it from your mail client or the portal.
22. **The word "still" in a rejection means an earlier question was never answered — and the reply channel is reactive-only.** Check whether a reply was ever posted before assuming the underlying technical issue is unresolved (these have different fixes). And you can't proactively post into a thread that doesn't exist yet — cancelling an old rejected submission to unblock a new one also closes off replying to that old thread, so post any prepared reply before you cancel, not after.
