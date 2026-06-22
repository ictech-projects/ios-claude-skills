# Network Security

## TLS & ATS

- All network requests must use HTTPS. Flag any `http://` URL in production code.
- App Transport Security (ATS) must not be disabled globally. Flag `NSAllowsArbitraryLoads: true` in `Info.plist` without a documented exception.
- Flag `NSExceptionAllowsInsecureHTTPLoads` for any domain other than a known development/staging host.
- TLS 1.2 is the minimum; TLS 1.3 is preferred. Flag `NSExceptionMinimumTLSVersion` set to `TLSv1.0` or `TLSv1.1`.

## Request Construction

- Never interpolate user input directly into URL strings — always use `URLComponents` with `queryItems`.
- Avoid logging full request URLs if they contain tokens or sensitive query parameters.
- Flag `URLSession` configurations with `waitsForConnectivity: false` on critical auth requests — this can silently fail on restricted networks.
- Set reasonable timeouts on all requests. Flag sessions with no timeout or timeouts exceeding 60 seconds for auth flows.
