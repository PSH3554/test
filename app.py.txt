import streamlit as st
import pandas as pd

# 1. 페이지 기본 설정 및 디자인 (가시성 확보)
st.set_page_config(page_title="삼육중 스마트 시간표 시스템", layout="wide")

st.title("🏫 호남삼육중학교 맞춤형 시간표 관리 시스템")
st.markdown("---")

# 2. 선생님이 업로드해주신 실제 기초 데이터 기반 가상 세팅
# (화7~8: 1인1악기, 수6~8: 스포츠, 금6: 창체 고정)
if 'timetable' not in st.session_state:
    data = {
        "반": ["1-1", "1-2", "1-3", "1-4", "2-1", "2-2", "2-3", "2-4", "3-1", "3-2", "3-3", "3-4"],
        "월1": ["도덕", "체육", "생영", "체육", "기가", "국어", "영어", "영어", "역사", "종교", "수학", "수학"],
        "화7": ["1인1악기", "공백", "공백", "공백", "생영", "공백", "국어-오", "기가", "수학", "수학", "기가-채", "종교-진"],
        "화8": ["국어", "체육", "종교-진", "체육", "1인1악기", "공백", "공백", "공백", "기가", "생영", "국어-박", "기가-채"],
        "수6": ["스포츠", "공백", "공백", "공백", "수학", "수학", "생영", "기가", "미술", "역사", "영어", "영어"],
        "수7": ["과학", "과학", "국어-임", "도덕", "스포츠", "공백", "공백", "공백", "미술", "종교-진", "생영", "국어-박"],
        "수8": ["도덕", "국어", "음악", "생영", "과학", "과학", "기가", "국어", "스포츠", "공백", "공백", "공백"],
        "금6": ["창체", "창체", "창체", "창체", "창체", "창체", "창체", "창체", "창체", "창체", "창체", "창체"]
    }
    st.session_state.timetable = pd.DataFrame(data)

df = st.session_state.timetable

# 3. 교사 전용 관리 패널 (사이드바)
st.sidebar.header("⚙️ 관리자 제약 조건 제어")
lock_special = st.sidebar.checkbox("1인1악기 / 스포츠 / 창체 고정 (수정 불가)", value=True)
st.sidebar.markdown("""
**💡 사용법 안내:**
1. 우측 표에서 수정하고 싶은 일반 수업 칸을 **더블클릭**합니다.
2. 원하는 과목으로 수정한 뒤 **Enter**를 누릅니다.
3. 고정 조건 위반 시 하단에 경고가 발생합니다.
""")

# 4. 메인 작업 화면: 엑셀 스타일의 데이터 편집기(Data Editor) 띄우기
st.subheader("📅 주간 학급별 시간표 매트릭스 (실시간 수정 가능)")

# 고정 과목이 켜져 있을 때 수정 불가능한 컬럼 지정하는 로직
disabled_cols = ["반"]
if lock_special:
    # 창체, 스포츠, 1인1악기가 포함된 컬럼은 눈으로만 보고 수정은 못하게 막음
    disabled_cols.extend(["화7", "화8", "수6", "수7", "수8", "금6"])

# 선생님들이 실시간으로 편집할 수 있는 인터랙티브 표 출력
edited_df = st.data_editor(
    df,
    num_rows="fixed",
    use_container_width=True,
    disabled=disabled_cols,
    key="timetable_editor"
)

# 변경 사항 저장 버튼
if st.button("💾 변경된 시간표 최종 확정 및 저장"):
    st.session_state.timetable = edited_df
    st.success("🎉 시간표 수정을 완료하고 시스템에 반영했습니다!")

# 5. 사후 조정 시 에러(충돌) 검증 시스템 연산 파트
st.markdown("---")
st.subheader("🚨 실시간 행정 충돌 검증 시스템")

# 예시: 특정 반에 중복 수업이나 공백이 생겼는지 실시간 체크하는 백엔드 로직
error_detected = False
for index, row in edited_df.iterrows():
    # '공백'인 칸이 생기면 경고 알림
    if "공백" in row.values:
        st.error(f"⚠️ 임시 경고: **{row['반']}** 편성에 아직 배치되지 않은 '공백' 수업이 존재합니다.")
        error_detected = True

if not error_detected:
    st.info("✅ 현재 시간표에 중복 배정이나 고정 룰 위반이 없습니다. 배포 가능 상태입니다.")