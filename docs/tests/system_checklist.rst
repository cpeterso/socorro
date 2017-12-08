======================
Socorro Test Checklist
======================

This is a high-level system-wide checklist for making sure Socorro is working
correctly in a specific environment. It's a helpful template for figuring out
what you need to change if you're pushing out a significant change.

**Note:** This is used infrequently, so if you're about to make a significant change,
you should go through the checklist to make sure the checklist is correct and
that everything is working as expected and fix anything that's wrong, THEN
make your change, then go through the checklist again.

Lonnen the bear says, "Only you can prevent production fires!"

Last updated: November 15th, 2017


How to use
==========

"Significant change" can mean any number of things, so this is just a template.
You should do the following:

1. Copy and paste the contents of this into a Google Doc, Etherpad, or
   whatever system you plan to use to keep track of status and outstanding
   issues.

2. Go through what you copy-and-pasted, remove things that don't make sense,
   and add additional things that are important. (Please uplift any changes
   via PR to this document that are interesting.)


Checklist
=========

::

    Migrations
    ==========

    Make sure we can run migrations

    * Django migrations

      Local dev environment:

      1. "docker-compose run webapp bash"
      2. "cd webapp-django/"
      3. "./manage.py showmigrations"

      -stage/-prod:

      1. See Mana

    * Alembic migrations

      Local dev environment:

      1. "docker-compose run processor bash"
      2. "alembic -c docker/config/alembic.ini current"

      -stage/-prod:

      1. See Mana


    Collector (Antenna)
    ===================

    Is the collector handling incoming crashes?

    * Check datadog Antenna dashboard for the appropriate environment.

      localdev: Check the logging in the console
      stage: https://app.datadoghq.com/dash/272676/antenna--stage
      prod: https://app.datadoghq.com/dash/274773/antenna--prod

    * Log into Sentry and check for errors.

    * Submit a crash to the collector. Verify raw crash made it to S3.


    Processor
    =========

    Is the processor process running?

    * Log into a processor node and watch the processor logs for errors.

      Log file: "/var/log/messages"

      To check for errors: "grep ERROR /var/log/messages | less"

    * Check Datadog "processor.save_raw_and_processed" for appropriate
      environment.

      localdev: Check the logging in the console
      stage: https://app.datadoghq.com/dash/272676/antenna--stage
      prod: https://app.datadoghq.com/dash/274773/antenna--prod

    Is the processor saving to ES? Postgres? S3?

    * Check Datadog
      "processor.es.ESCrashStorageRedactedJsonDump.save_raw_and_processed.avg"

      stage: https://app.datadoghq.com/dash/272676/antenna--stage
      prod: https://app.datadoghq.com/dash/274773/antenna--prod

    * Check Datadog
      "processor.s3.BotoS3CrashStorage.save_raw_and_processed" for
      appropriate environment.

      stage: https://app.datadoghq.com/dash/272676/antenna--stage
      prod: https://app.datadoghq.com/dash/274773/antenna--prod

    * Check Datadog
      "processor.postgres.PostgreSQLCrashStorage.save_raw_and_processed"

      stage: https://app.datadoghq.com/dash/272676/antenna--stage
      prod: https://app.datadoghq.com/dash/274773/antenna--prod


    Submit a crash or reprocess a crash. Wait a few minutes. Verify the crash was
    processed and made it to S3, ES and Postgres.

    **FIXME:** We should write a script that uses envconsul to provide vars and takes
    a uuid via the command line and then checks all the things to make sure it's
    there. This assumes we don't already have one--we might!


    Webapp
    ======

    Is the webapp up?

    * Use a browser and check the healthcheck (/monitoring/healthcheck)

      It should say "ok: true".

    Is the webapp throwing errors?

    * Check Sentry for errors
    * Log into webapp node and check logs for errors.

      Log file: "/var/log/messages"

      To check for errors: "grep ERROR /var/log/messages | less"

    * Run QA Selenium tests.

      localdev: ?
      stage: In IRC: "webqatestbot build socorro.stage.saucelabs"
      prod: In IRC: "webqatestbot build socorro.prod.saucelabs"

    Can we log into the webapp?

    * Log in and check the profile page.

    Is the product home page working?

    * Check the Firefox product home page (/ redirects to /home/product/Firefox)

    Is super search working?

    * Click "Super Search" and make a search that is not likely to be cached.
      For example, filter on a specific date.

    Top Crashers Signature report and Report index

    1. Browse to Top Crashers
    2. Click on a crash signature to browse to Signature report
    3. Click on a crash id to browse to report index

    Can you upload a symbols file?

    * Download https://github.com/mozilla/socorro/blob/master/webapp-django/crashstats/symbols/tests/sample.zip
      to disk
    * Log in with a user with permission to upload symbols.
    * Go to the symbol upload section (/symbols/upload/web)
    * Try to upload the "sample.zip" file.
    * To verify that it worked, go to the public symbols S3 bucket and check
      that there is a "xpcshell.sym" file in the root with a recent modify
      date.


    Crontabber
    ==========

    Is crontabber working?

    * Check healthcheck endpoint (/monitoring/crontabber/)

      It should say ALLGOOD.

    * Check the webapp crontabber-state page (/crontabber-state/)

    Is crontabber throwing errors?

    * Check Sentry for errors
    * Log into admin node and check logs for errors

      Log file: "/var/log/socorro/crontabber"

      To check for errors: "grep ERROR /var/log/messages | less"


    Stage submitter
    ===============

    Is the stage submitter running and sending crashes?

    * Check Datadog dashboard for Antenna on -stage