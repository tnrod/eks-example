# Claude Skills 사용 가이드

## 설치된 스킬

### 📄 pdf-report
PDF 보고서 작성을 위한 전문 스킬입니다.

**기능:**
- 자동 목차 생성
- 일관된 보고서 양식
- 하단 서명 섹션 포함
- PDF 변환 가이드 제공

**사용 방법:**
```
/pdf-report
```

또는 Claude에게 다음과 같이 요청하세요:
- "PDF 보고서를 작성해줘"
- "보고서 양식으로 문서를 만들어줘"

## 서명 이미지 설정

보고서 하단에 서명을 추가하려면:

1. 서명 이미지 파일 준비 (PNG 권장)
2. 파일명: `signature.png`
3. 권장 크기: 200x80px
4. 투명 배경 권장

### 서명 이미지 생성 방법

**온라인 도구:**
- https://www.signwell.com - 온라인 서명 생성
- https://signature-generator.com - 무료 서명 생성기
- Canva - 커스텀 서명 디자인

**직접 생성:**
1. 종이에 서명 작성
2. 스캔 또는 사진 촬영
3. 배경 제거 (remove.bg 등)
4. PNG로 저장

## 보고서 작성 예시

```bash
# 1. Claude에게 요청
"월간 보고서를 PDF 양식으로 작성해줘. 제목은 '2025년 10월 프로젝트 현황'이야"

# 2. 생성된 마크다운을 PDF로 변환
pandoc report.md -o report.pdf --toc --pdf-engine=xelatex
```

## 커스터마이징

스킬을 수정하려면:
```bash
# 스킬 파일 편집
nano .claude/skills/pdf-report.md

# 또는
code .claude/skills/pdf-report.md
```

## 문제 해결

**Q: 서명 이미지가 표시되지 않아요**
- 이미지 경로 확인: 문서와 같은 디렉토리에 있는지 확인
- 상대 경로 사용: `./signature.png` 또는 `../images/signature.png`

**Q: PDF 변환이 안 돼요**
- pandoc 설치: `sudo apt-get install pandoc`
- xelatex 설치: `sudo apt-get install texlive-xetex`

**Q: 목차 링크가 작동하지 않아요**
- 섹션 제목의 앵커 이름 확인
- 공백은 `-`로, 특수문자는 제거됨
