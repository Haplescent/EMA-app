# EMA-app

Desired Features
================

-   participants receive another notification after a set amount of time if they did not click the link
-   Custom scheduling
    -   burst scheduling
    -   participant triggered
-   API access to construct and edit links and schedule for a study
    -   e.g. if future survey links or timing depends on responses to study, researcher can create a program which processes survey responses and edits scheduling or links
-   one study can contain "types" of messages which allows people to create different parameters for things like diary vs EMA
-   send any type of ping to a participant as a test

Database structure
==================

Tables
------

-   researchers
    -   researcher_id
    -   name
    -   email
    -   api keys for messaging service
-   participants
    -   participant_id
-   studies
    -   study_id
    -   researcher_id
-   ping_type
    -   this database holds a template for pings that will be scheduled. when the researcher creates a study, they will create as many ping typeses as they need for that study which then get assigned to participants when they are added. the ping_type doesn't have any particular participant info. Instead it just contains default parameters.
    -   ping_type_id
    -   study_id
    -   researcher_id
    -   message
    -   ping_trigger_type: {scheduled, participant triggered, both}
    -   survey URL skeleton
        -   base URL and placeholder key value variables to be filled in per message
    -   mean times and jitter SD (JSON schedule for one day of pings)
    -   every_x_days
-   pings
    -   this database holds the pings scheduled to be sent or already sent. Once a ping makes it in this db it will be sent according to the timestamp.
    -   ping_id
    -   ping_type_id
    -   participant_id
    -   researcher_id
    -   study_id
    -   message
    -   send_timestamp
    -   expire_timestamp
    -   sent_status
        -   indicates whether the message was sent successfully based on API response code
    -   url_vars
        -   a JSON of key value pairs which will be appended to the URL.
        -   can dynamically include participant_id and expire_timestamp
    -   survey URL
    -   redirect code
        -   this is a long and complex code which is used to create the participant facing link. the participant gets sent a link to our server which contains this redirect code as a query variable and then they get routed to the correct survey. This allows us to track when a participant has clicked a link or not.
    -   clicked_status
        -   indicates whether URL was clicked and redirect was successfully implemented

Application
===========

Interface
---------

### Pages

-   researcher login
-   view all studies
    -   for a particular researcher
-   create new study
    -   adds study to study database with global study settings
-   design pings
    -   create and edit the ping types for a given study
-   view study schedule
    -   see the whole list of scheduled pings for a study
-   add participant to a study
    -   adds a new participant to the database
    -   schedule them for pings based on the ping types for that study
    -   send test messages of each type of ping types
-   view participants for a study
    -   view participant IDs
    -   can filter by study id
-   redirect page
    -   this is a page with no content. the participant receives links to this page with a unique query variable which identifies which ping the link is associated with. The redirect page then forwards them to the survey and records their click.

### Classes and methods

-   Study
    -   add()
        -   adds a new study to study DB
-   PingType
    -   add()
        -   args: study_id, message, trigger_style, schedule_dict
        -   adds a new ping type to the DB
        -   example of what schedule dict looks like
            -   {pings: {mean_time: 8:00:00, jitter_sd: 2}, {mean_time: 16:00:00, jitter_sd: 2}}
-   Ping
    -   send()
        -   args:
        -   pushes the ping whichever platform
-   Participant
    -   add()
        -   args: participant_id, study_id
        -   adds a new participant to participant DB
    -   remove()
        -   args: participant_id, study_id
-   Researcher
    -   add()
        -   args: name, email, api_key
        -   adds a new researcher to researcher DB
        -   calls generate_new_id() to generate an unused ID
    -   generate_new_id()
        -----------------

    -   remove()
-   Redirect
    -   record_click()
        -   args: message_id
    -   redirect_forward()
        -   args: destination_url