Tailscale Installation
Install Tailscale

curl -fsSL https://tailscale.com/install.sh | sh

Authenticate

sudo tailscale up

Verify

tailscale status

Check Assigned IP

tailscale ip -4
