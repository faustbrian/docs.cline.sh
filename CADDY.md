⏺ Astro builds to dist/ by default. Here's your Caddyfile:

  docs.cline.sh {
      root * /path/to/docs.cline.sh/dist
      encode gzip
      file_server

      # Handle client-side routing
      try_files {path} {path}/ /404.html
  }

  Replace /path/to/docs.cline.sh with your actual project path.

  For automatic builds on changes:

  docs.cline.sh {
      root * /path/to/docs.cline.sh/dist
      encode gzip
      file_server
      try_files {path} {path}/ /404.html

      # Optional: Add security headers
      header {
          X-Content-Type-Options nosniff
          X-Frame-Options DENY
          Referrer-Policy no-referrer-when-downgrade
      }
  }

  Then reload: caddy reload
