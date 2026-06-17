# remote_control_with_zenoh
allow a remote control of a rele through zenoh using TLS as encription

## TLS
https://zenoh.io/docs/manual/tls/

## CA certificate
### MiniCA
https://github.com/jsha/minica
or
### Let's encrypt
https://zenoh.io/blog/2023-04-04-letsencrypt/

## Zenoh-pico
https://zenoh-pico.readthedocs.io/en/latest/config.html#tls

### compile time *TOCHECK*
#### CMake
```bash
cmake -DZ_FEATURE_LINK_TLS=ON ..
cmake --build .
```
or
#### manual
```c
#define Z_FEATURE_LINK_TLS 1
```

### code pico
```c
#include <zenoh-pico.h>

int main() {
    z_owned_config_t config = z_config_default();

    // 1. Set your endpoint to use the 'tls/' protocol
    z_config_insert_str(z_config_loan(&config), Z_CONFIG_CONNECT_KEY, "tls/localhost:7447");

    // 2. Point to your Root CA certificate file
    z_config_insert_str(
    z_config_loan(&config), 
    Z_CONFIG_TLS_ROOT_CA_CERTIFICATE_BASE64_KEY, 
    "MIIBiTCCATKgAwIBAgIUW5zV..." // Your Base64 string here
);

    // 3. Open the Zenoh session
    z_owned_session_t session = z_open(z_config_move(&config));
    if (!z_session_check(&session)) {
        // Handle connection failure
        return -1;
    }

    // Your zenoh-pico code goes here...

    z_close(z_move(session));
    return 0;
}
```

### router
```json
{
  mode: "router",
  listen: {
    endpoints: ["tls/0.0.0.0:7447"]
  },
  transport: {
    link: {
      tls: {
        server_private_key: "/home/user/tls/server.key",
        server_certificate: "/home/user/tls/server.crt"
      }
    }
  }
}
```
