 -- Oct 28, 2024 --
# Enabling SSH key for SSH agent
After generating a SSH key with (or importing to) 1password you need to include it inside of 1password's SSH agent config file, which is most likely located at `/home/michel/.config/1Password/ssh/agent.toml`.

List each SSH you want as follows:

```
[[ssh-keys]]
item = "gitlab_company_key"
vault = "company"

[[ssh-keys]]
item = "github_private_key"
vault = "Private"
```