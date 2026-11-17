# Ember Media Manager 1.11.x — TheTVDB title-search community hotfix

Unofficial community hotfix for Ember Media Manager 1.11.x on Windows.

It fixes two related problems:

1. searching a TV show by title returns no results, while entering a TVDB ID manually still works;
2. after selecting a newly found show, Ember can throw `NullReferenceException` in `TVDB.Model.SeriesDetails.DeserializeActors()`.

The movie scrapers are not changed.

## Root cause

The bundled `TVDB.dll` calls the legacy endpoint:

```text
http://thetvdb.com/api/GetSeries.php?seriesname=...&language=...
```

As of 16 November 2026, that endpoint returns HTTP 200 with an empty XML document:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Data></Data>
```

The hotfix uses the same unauthenticated search service currently used by TheTVDB's public website, restricts results to series, preserves localized titles, forces TLS 1.2, applies a 20-second timeout and limits JSON parsing to 5 MiB.

The XML readers were also made tolerant of empty `actors.xml`, `banners.xml` and series documents.

## Important status note

`https://api4.thetvdb.com/web/search/queries` is a website-internal endpoint, not the officially supported TheTVDB v4 API. It may change or disappear without notice. Treat this as a best-effort community hotfix. A durable upstream solution should migrate Ember to the official authenticated v4 API after its licensing/key model has been agreed with the maintainer and TheTVDB.

## Tested configuration

- Windows 11 x64
- Ember Media Manager 1.11.1
- `TVDB` assembly identity retained as version `1.2.0.0`
- Search: `Margo Ha Problemi Di Soldi` → `Margo ha problemi di soldi`, TVDB ID `445651`
- Details: localized title plus 9 actors; no exception
- Additional searches: `Lost`, `VisionQuest`, `Il commissario Montalbano`

## Package contents

- `thetvdbapi-ember-title-search.patch` — unified patch against `VDSoft/TheTVDBApi` master
- `source/` — complete corresponding TheTVDBApi source tree used for the binary
- `tests/` — small integration-test harnesses
- `binary/TVDB.dll` — optional prebuilt community binary for the tested configuration
- `LICENSE` — GPL-3.0 license inherited from TheTVDBApi
- `FORUM_POSTS.md` — ready-to-edit English, German and Italian announcements
- `ISSUE_AND_PR.md` — GitHub issue and pull-request drafts
- `SHA256SUMS.txt` — checksums

No personal API key, PIN, token, database, configuration, media path or log is included. Whenever an API credential must be mentioned, the placeholder is `APIXXXXXXXX`.

## Safe installation of the optional binary

1. Close Ember completely.
2. Back up these two files:
   - `Modules\scraper.data.tvdb\TVDB.dll`
   - `Modules\scraper.image.tvdb\TVDB.dll`
3. Copy `binary\TVDB.dll` into both directories.
4. Start Ember and test a title search.

Do not replace `scraper.Data.TVDB.dll`, `scraper.Image.TVDB.dll`, the movie scrapers, Ember's database or profile configuration.

## Build from source

Requirements:

- Visual Studio/MSBuild with the .NET Framework 4.5 targeting pack
- the GPL-3.0 source from `https://github.com/VDSoft/TheTVDBApi`

Apply the patch from the root of a clean TheTVDBApi checkout:

```text
git apply thetvdbapi-ember-title-search.patch
```

Build `source\src\TVDB\TVDB.csproj` in `Release|Any CPU`, preferably with debug symbols disabled for a redistributable binary, then copy the resulting `TVDB.dll` to the two TVDB module directories listed above.

Ember's solution expects TheTVDBApi as a sibling checkout (`..\TheTVDBApi`). The currently active Ember fork is `https://github.com/nagten/Ember-MM-Newscraper`.

## Rollback

Close Ember and restore the two original `TVDB.dll` files from your backup.

## License and attribution

TheTVDBApi is licensed under GPL-3.0. This package includes the license, modified source files and a reproducible patch. The original project is credited to VDSoft/Christoph van der Fecht. Ember Media Manager remains the work of its respective contributors.

This project is not endorsed or certified by TheTVDB.
