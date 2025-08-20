## Couldn't connect to GitHub from local machine after hitting error 
## no such identity: /Users/--/.ssh/id_ed2551: No such file or directory

# FIX:
## Check
ssh-add -l

## If 'agent has no identities'

ssh-add ~/.ssh/id_ed25519 #can be ed2551x check cat ~/.ssh/config for the right ssh id
