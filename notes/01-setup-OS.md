# Setup
Installed Raspberry Pi OS using Raspberry Pi Imager from:

https://www.raspberrypi.com/software/?utm_source=chatgpt.com

- connected to Pi via SSH
- fixed locale issue (en_GB.UTF-8)

## Problem: Locale warning
When I connected via SSH, I saw warnings like:
```
WARNING! Your environment specifies an invalid locale
The unknown environment variables are:
   LC_CTYPE=en_US.UTF-8 LC_MESSAGES=en_US.UTF-8 LC_ALL=en_US.UTF-8
```

## Solution
#### 1. Install the locales by running:
```
   sudo dpkg-reconfigure locales
```
selected: en_GB.UTF-8

#### 3. Set default locale:
```
   sudo update-locale LANG=en_GB.UTF-8 LC_ALL=en_GB.UTF-8
```

#### 4. Fixed LC_CTYPE:
```
   sudo update-locale LC_CTYPE=en_GB.UTF-8
```

#### 5. Reconnected via SHH
No more warnings after login. Locale must be both generated and set as default
