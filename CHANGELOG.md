# Changelog

## v1.1.3

### Fix and enhancements

- Fix nginx_daemon_enabled dynamic default value

## v1.1.2

### Fix and enhancements

- Setup dependencies only if action is setup

## v1.1.1

### Fix and enhancements

- Cleanup local source directory

## v1.1.0

### Minor compatibility breaks

- Add cleanup action and `nginx_role_action` variable (required)

### Features

- Add container daemon mode (no daemon :))

### Fix and enhancements

- Can disable become (e.g. when building a container)
- Install build packages only if required
- Install less packages to build nginx
- Split packages into two categories (clone, build)

## v1.0.0

- Initial release
