on:
  push:
    paths:
      - '**.png'
      - '**.jpg'

jobs:
  build-gallery:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Generate Thumbnail Grid in README
        run: |
          echo "## 🖼️ All Uploaded Assets" > README.md
          echo "" >> README.md
          echo "<div style='display: flex; flex-wrap: wrap; gap: 10px;'>" >> README.md
          find . -type f \( -iname "*.png" -o -iname "*.jpg" \) ! -path "./.git/*" | sort | while read img; do
            clean_path="${img#./}"
            filename=$(basename "$clean_path")
            echo "<div style='display:inline-block; text-align:center; margin:6px; border:1px solid #ddd; padding:6px; border-radius:6px;'>" >> README.md
            echo "  <a href='$clean_path'><img src='$clean_path' width='120' height='120' style='object-fit:contain;' /></a><br/>" >> README.md
            echo "  <sub><b>$filename</b></sub>" >> README.md
            echo "</div>" >> README.md
          done
          echo "</div>" >> README.md

      - name: Commit updated README
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git diff --quiet && git diff --staged --quiet || (git commit -m "Update image thumbnails" && git push)
