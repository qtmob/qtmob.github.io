---
layout: site
title: The Making of a Touristic App - How To
permalink: touristic-app-how-to
contributors: [marco_piccolino]
level: intermediate
---
[DRAFT] Level: {{page.level}}

*First published: 2015-10-01;* *Contributors:* {% for contributor in page.contributors %} -- {{ site.authors[contributor].name }} {% endfor %}

**Problem:** you want to come up with a manageable structure for a touristic app.

**Solution:** you define responsibilities of each application layer and component clearly.

## Application layers

### Page navigation layer

To navigate between pages, we use a state machine. The page navigation layer only knows about pages and their api.

### Page layer

The page layer defines what templates (a placeholder and a finished) and what data are associated to the page.

### Page template layer

The page template layer define the structure of the page (components) and the internal relationships between its components.

If you have any suggestions for improvements on the above technique, come join the discussion at [QtMob/Slack](https://qtmob.slack.com) (ask for an [invite](mailto:marco.a.piccolino@gmail.com) first).
