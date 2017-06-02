# Audience

For meteor projects - while using mongo.

# Version
Mongo: 3.2
# Use Case
You want to populate users with password (mimicing accounts-password) in Meteor.

# Example Script

Say you have an environment variable USERS=joe:password,john:anotherpassword

Your script could be as follows.(You need to volume map - './init.sh:/docker-entrypoint-initdb.d/init.sh:ro')

```
#!/bin/bash
# shellcheck disable=SC2154

# bash script mode - strict
# -e standard bash does not do this. It instructs bash to immediately exit if any command has a non-zero exit status
# -u affects variables. Reference to any variable you haven't previously defined will fail script with the exceptions.
# -o pipefail - shows the line number and details - like sourcemap in nodejs
# IFS - Internal Field Separator - gives the ability to loop through say comma separated env
set -euo pipefail
IFS=$','
# check for null or empty for variables.
users=${USERS:-""}

if [[ -z ${users} ]] ; then
  for x in $users; do
    password=$(echo "abc:def" | awk -F':' '{print $1}')
    encrypted=$(bcrypt $(echo -n "$password" | sha256sum | cut -d " " -f 1))
    "${mongo[@]}" "$MONGO_INITDB_DATABASE" <<-EOJS
            //you have the password variable now,
            //create the user object in the way you want.
            var user={}
            db.users.insert(user);
      EOJS
  done
fi
```
