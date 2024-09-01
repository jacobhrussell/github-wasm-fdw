# GitHub Wasm FDW

This example reads the [realtime GitHub events](https://api.github.com/events) into a Postgres database. 

## Project Structure

```bash
├── src
│   └── lib.rs              # The package source code. We will implement the FDW logic, in this file.
├── supabase-wrappers-wit   # The Wasm Interface Type provided by Supabase. See below for a detailed description.
│   ├── http.wit
│   ├── jwt.wit
│   ├── routines.wit
│   ├── stats.wit
│   ├── time.wit
│   ├── types.wit
│   ├── utils.wit
│   └── world.wit
└── wit                     # The WIT interface this project will use to build the Wasm package.
    └── world.wit
```

A [Wasm Interface Type](https://github.com/bytecodealliance/wit-bindgen) (WIT) defines the interfaces between the Wasm FDW (guest) and the Wasm runtime (host). For example, the `http.wit` defines the HTTP related types and functions can be used in the guest, and the `routines.wit` defines the functions the guest needs to implement.

## Getting started

To get started, visit the [Wasm FDW developing guide](https://fdw.dev/guides/create-wasm-wrapper/).

## Usage

To use this Wasm foreign data wrapper on Supabase, create a foreign table like below,

```sql
create extension if not exists wrappers with schema extensions;

create foreign data wrapper wasm_wrapper
handler wasm_fdw_handler
validator wasm_fdw_validator;

create server example_server
foreign data wrapper wasm_wrapper
options (
    fdw_package_url 'https://github.com/${GITHUB_REPOSITORY}/releases/download/v${VERSION}/${PROJECT}.wasm',
    fdw_package_name '${PACKAGE}',
    fdw_package_version '${VERSION}',
    fdw_package_checksum '${CHECKSUM}',
    api_url 'https://api.github.com'
);

create schema github;

create foreign table github.events (
id text,
type text,
actor jsonb,
repo jsonb,
payload jsonb,
public boolean,
created_at timestamp
)
server example_server
options (
    object 'events',
    rowid_column 'id'
);
```

For more details, please visit https://fdw.dev.