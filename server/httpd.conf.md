## Server Hardening
ServerSignature Off
ServerTokens Prod
FileETag None
Header set X-XSS-Protection "1; mode=block"
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
