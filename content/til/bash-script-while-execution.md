+++
title = "Bash scripting"
date = 2023-03-12
updated = 2023-03-12
type = "post"
draft = true
description = "How is a bash script intrepreted"
in_search_index = true
[taxonomies]
TIL-Tags = ["gitlab", "CI/CD"]
+++

You can edit a bash script while it is running.
Example: if you execute a bash script and can add a new command towards the end of the script.....the new command will be executed if the program hasn't exited already.
Something like this should affect all interpreted languages. However, this can't happen in Py because there is a pre-compilation (to Pyc) step before interpretation.