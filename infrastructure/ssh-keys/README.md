# SSH Public Keys

Place your team members' SSH public keys in this directory.

## Adding Keys

1. Each team member should add their public key file here (e.g., `member1.pub`, `member2.pub`)
2. The files should contain the public key in standard format:
   ```
   ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC... user@hostname
   ```

## Example

To get your public key:
```bash
cat ~/.ssh/id_rsa.pub
```

Or generate a new one:
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Then copy the `.pub` file content into a new file here.

## Required Keys

Add at least 2 team member public keys for the assignment requirements.
