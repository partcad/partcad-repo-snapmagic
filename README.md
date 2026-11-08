# SnapMagic for PartCAD

SnapMagic Search (formerly SnapEDA) 3D models as PartCAD parts, published with the other
collections of models as `//pub/collection/snapmagic`. Its one external dependency, `models`,
is served by the `snapmagic_repo` plugin:

* **a part is a manufacturer's part**, named by its part number and manufacturer -
  `LM358DR~texas-instruments`, since two manufacturers can share a part number - with
  SnapMagic's STEP model of it as its geometry;
* **it says how to buy it**: its `vendor` is the manufacturer and its `sku` the part number,
  which is what a store quoting by manufacturer and part number - DigiKey's
  (`//pub/svc/commerce/digikey`), Octopart's - prices;
* **nothing is listed**: a part is asked for by name, and found with `pc search parts`, which
  asks SnapMagic's own search.

DigiKey's catalogue draws its parts with these models where DigiKey has no model file of its
own: most of the "EDA / CAD Models" DigiKey lists for a product are SnapMagic pages, and such a
part is an alias of `//pub/collection/snapmagic/models:<MPN>~<manufacturer>`.

## Setting it up

Downloading a model takes a SnapMagic account, which is free:

```shell
pc login //pub/collection/snapmagic:snapmagic_repo
```

asks for its user name (or email) and password, exchanges them for a token once - as
SnapMagic's own CAD plugins for KiCad and Fusion do - and keeps only the token, in
`~/.partcad/config.yaml`, under `parameters`, by the plugin's name. `SNAPMAGIC_TOKEN` works too,
where the plugin's sandbox sees the environment. An account made by signing in with Google
needs a password set first, on snapeda.com's account page.

## Using it

```shell
pc search parts -P //pub/collection/snapmagic/models -k LM358DR
pc info //pub/collection/snapmagic/models:LM358DR~texas-instruments
pc supply quote --provider //pub/svc/commerce/digikey:digikey \
  "//pub/collection/snapmagic/models:LM358DR~texas-instruments#10"
```

A part number two manufacturers have needs the manufacturer (`LM358DR` is made by Texas
Instruments and by Rohm); a manufacturer is known by its name as SnapMagic spells it
(`ON Semiconductor` is `on-semiconductor`) and by the one in its page's address (`onsemi`).

## How SnapMagic is asked

SnapMagic offers an official API on request, at <https://www.snapeda.com/get-api/>, whose
documentation is SnapMagic's to give; access has been asked for. Until it is in hand, what this
asks is what SnapMagic's own CAD plugins ask:

| What PartCAD asks | SnapMagic |
| --- | --- |
| logging in | `POST /account/api-login/` with the user name and password: `{"status": "logged_in", "token"}` |
| a search, a part | `GET /api/v1/parts/search?q=<MPN>&token=...`: results with `part_number`, `manufacturer`, `has_3d`, `package`, ... |
| its model | `GET /api/v1/parts/download_part?part_number&manufacturer&...&format=step&token=...`: a link that lasts minutes, to the STEP file or a zip holding it; never kept |

A failure is a 200 with `error` or `status` in it, which is said in words and not kept:
`not logged in` and an expired token say to log in again, `unverified_email` to verify the
account, `download limit exceeded` to try again later, `No models for this part` that there
is none.

## What to know

* **SnapMagic's terms ask for its written permission before any automated access**, which is
  what the request for an official API is for; rely on this once that is answered.
* **Every model is SnapMagic's**, under CC BY-SA 4.0 with its Design Exception: fetched for you,
  as you need it, into your own PartCAD cache, and never put into a package.
* SnapMagic publishes no rate limits; a search is one request, and so is a part.

Not yet run against SnapMagic itself, for want of an account here: the tests answer it in the
shapes its API answered without credentials, and the package was run end to end in PartCAD
0.9.22, through DigiKey's aliases, against a server standing in for it (the plugin's `api`
parameter points it there).

## Tests

```shell
pip install pytest
pip install --no-deps partcad    # for plugin_models, which PartCAD gives every plugin
pytest
```
