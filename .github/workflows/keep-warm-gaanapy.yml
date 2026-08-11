# =============================================================================
# .github/workflows/keep-warm.yml
# =============================================================================
# Destination: this GaanaPy repo (github.com/paragairun/GaanaPy)
#   .github/workflows/keep-warm.yml
#
# Pings this service's own Render deployment every 10 minutes to prevent
# Render's free-tier from spinning it down after 15 minutes of inactivity.
# Lives in this repo (not the karaoke app's repo) since it's this service
# keeping itself alive, not something the karaoke app needs to know about.
#
# WHY 10 MINUTES: Render's free tier spins down after ~15 min with no
# incoming requests. Pinging every 10 min means it never goes 15 minutes
# without traffic, so it never idles down in the first place.
#
# WHY THE ROOT PATH IS ENOUGH: Render's idle timer resets on ANY incoming
# HTTP request, regardless of response status code. This app's root route
# ("/") already returns a clean 200 with {"Docs": "/docs", ...} — confirmed
# working directly against this exact URL — so no dedicated health
# endpoint is needed.
#
# KNOWN LIMITATION, stated honestly: GitHub Actions' schedule trigger is
# NOT guaranteed to fire at the exact minute — under GitHub's own load,
# scheduled runs can occasionally slip by a few minutes. This should fully
# solve cold starts in practice, but isn't a 100% ironclad guarantee the
# way upgrading to Render's paid tier would be.
#
# IF THIS SERVICE'S URL CHANGES: update the URL below. This currently
# points at the Singapore-region deployment (gaanapy-sg.onrender.com),
# not the earlier Virginia one -- the Virginia service is being retired.
# =============================================================================

name: Keep Service Warm

on:
  schedule:
    - cron: '*/10 * * * *'
  workflow_dispatch: # lets you manually trigger a ping from the Actions tab to test

jobs:
  ping:
    runs-on: ubuntu-latest
    steps:
      - name: Ping this service on Render
        run: |
          # --max-time 60: a cold start can take 30-50s to respond. We wait
          # long enough for the wake-up to actually complete rather than
          # timing out mid-boot and having to wait for the next cron tick.
          #
          # No --fail flag: any HTTP response means the request reached
          # the server and reset its idle timer -- that IS success for our
          # purposes, regardless of status code.
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" --max-time 60 \
            "https://gaanapy-sg.onrender.com/" || echo "no-response")

          echo "Ping response: $STATUS"

          if [ "$STATUS" = "no-response" ]; then
            echo "::warning::No response at all (connection-level failure) -- worth checking if the service is actually down, not just cold."
          else
            echo "Ping reached the server successfully (HTTP $STATUS) -- idle timer reset regardless of status code."
          fi
