# Event Tracking Guide

## Basics of web tracking

Look at the user tracking in context of an open session. Anonymous visitor visits your webiste and browses your content. You may want to capture user activity by tracking events, updating user attributes and identifying user by unique ID in the right moment (e.g. sign up form submit).

### Identify users
By default Exponea tracks users only by cookie id. The identify function assings current user given id or ids. Use the same id in import wizard to match users.

    multiple_identifiers = {
      'registered': 12345, 
      'email': 'user@domain.com'
    };
    exponea.identify(multiple_identifiers)
  
### Update user attributes
Create or update user attributes. Use attributes for storing static data (age, gender, location). Timestamp of update isn't persisted.

    user_attributes = {
      email: 'john@rambo.com',
      property2: 123
    };
    exponea.update(user_attributes)
  
### Track custom events
Adds new event to profile of current user. Events contain timestamp and are better for analyses than user attributes.

    event_attributes = {
      property1: 'value1',
      property2: 123
    };
    exponea.update(user_attributes)
  
