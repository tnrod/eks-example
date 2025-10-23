# 서명 이미지 생성 가이드

PDF 보고서에 서명을 추가하려면 `signature.png` 파일이 필요합니다.

## 빠른 시작

### 옵션 1: 온라인 서명 생성기 사용

1. https://www.signwell.com/online-signature/draw/ 방문
2. 서명 작성 또는 타이핑
3. PNG 형식으로 다운로드
4. 파일명을 `signature.png`로 변경
5. 프로젝트 루트 디렉토리에 저장

### 옵션 2: 직접 생성

```bash
# ImageMagick을 사용한 간단한 텍스트 서명
convert -size 200x80 xc:white -font Arial -pointsize 24 \
  -gravity center -annotate +0+0 "Your Name" \
  signature.png
```

### 옵션 3: SVG를 PNG로 변환

```bash
# Inkscape 사용
inkscape signature.svg --export-png=signature.png --export-width=200

# rsvg-convert 사용
rsvg-convert -w 200 -h 80 signature.svg -o signature.png
```

## 권장 사양

- **형식:** PNG (투명 배경 권장)
- **크기:** 200x80 픽셀 (또는 비슷한 비율)
- **배경:** 투명 또는 흰색
- **파일명:** `signature.png`
- **위치:** 프로젝트 루트 디렉토리

## 서명 이미지가 없을 때

서명 이미지가 없어도 보고서는 정상적으로 작동합니다.
단, PDF 변환 시 이미지 경로 오류가 표시될 수 있습니다.

임시로 이미지 참조를 제거하려면 `CODE_REPORT.md`에서
다음 라인을 주석 처리하세요:

```markdown
<!-- ![서명](./signature.png) -->
```
