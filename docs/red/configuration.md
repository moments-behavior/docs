# Configuration

By default, `red` saves projects to `$HOME/red_data` and prompts you to navigate to a media folder each time you open a video.

To override either, create `~/.config/red/config.json`:

```json
{
  "media_folder": "/nfs/exports/ratlv",
  "project_folder": "/nfs/exports/ratlv/fetch_runs"
}
```

Recognized keys:

- `media_folder` — default folder shown in the "open videos" dialog.
- `project_folder` — default save location for `.redproj` projects.
