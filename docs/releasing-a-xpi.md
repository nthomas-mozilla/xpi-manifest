# Releasing a xpi

## Release phases

### Build

In the build phase, we generate a release build, a dep-signing (development signing) task, and a test task. Additionally, there's a notify task that runs at the start of the graph, and a second notify task that runs after the signing task runs.

We trigger the build graph to generate a release build for QA to test.

### Promote

In the promote phase, we release-sign the build generated in the build graph. Because we're release-signing the same binary generated in the build graph, we're shipping the bits that we've tested (as opposed to rebuilding and release-signing the new build). There are also `release-notify-started` and `release-notify-completed` tasks that notify via email.

In the future, we could add additional tasks here, or additional phases, as needed. These could, for example, push to S3, push to XPIHub, push to bouncer, push to AMO, push to balrog, version bump the xpi.

## Configuring email notifications

These are configured per addon-type (privileged webextension vs system addon) and per phase (build or promote), in the xpi manifest repo's [`taskcluster/ci/config.yml`](../taskcluster/ci/config.yml), under `release-promotion.notifications`.

You may want to add a +comment (e.g. `email+system-addon-release-builds@m.c` for easier filtering.

## Kicking off the release

We'll use ship-it for this. This isn't quite ready yet. You'll need LDAP perms and VPN to reach it.

When we want a release build:

  - Connect to VPN
  - Connect to ship-it
  - Choose XPI out of the projects to build
  - Choose a xpi manifest revision. You probably want the latest.
  - Choose a xpi to build. Currently private-repo xpis aren't available, but we'll add support for them.
  - Choose a source repo revision. You probably want the latest.
  - Click `Doo Eet`. This will create the release, but not schedule it.
  - Click the `Build` on the progress bar for the appropriate row (this will be labeled with the XPI name, version, build number).
  - Once the build graph is scheduled, the `Build` will be a link to the build graph.
  - Once the build graph is finished, the first half of the progress bar will be green.
  - QA can then download the build from the build task (unsigned) or the dep-signing task (signed with the developer key).

At this point QA, developers, or project managers may request changes.

If we need a new release build:

  - Cancel the previous release
  - Repeat the above steps, but for a build `n+1`.

When we're ready to sign off on shipping this xpi:

  - Click on the `Promote` portion of the progress bar, and sign off.
  - When we have the quorum of signoffs, we'll schedule the promote graph, and we'll get a release-signed xpi.
