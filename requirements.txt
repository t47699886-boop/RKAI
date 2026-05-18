import streamlit as st
from google import genai
import requests

# 페이지 제목 및 레이아웃 설정
st.set_page_config(page_title="RKAI 플랫폼", page_icon="🤖", layout="wide")

st.title("🤖 RKAI 인공지능 플랫폼 대시보드")
st.caption("Google Gemini AI 및 Slack 알림 시스템이 연동된 통합 제어 센터입니다.")

# 1. secrets.toml 파일에서 보안 키 안전하게 로드
try:
    gemini_key = st.secrets["GEMINI_API_KEY"]
    slack_url = st.secrets["SLACK_WEBHOOK_URL"]
except Exception as e:
    st.error("🚨 secrets.toml 파일에서 API 키를 읽어오지 못했습니다. 파일 위치와 오타를 확인해주세요.")
    st.stop()

# 2. 메인 AI 기능 화면 구현
st.header("💡 제미나이 AI 어시스턴트")
user_input = st.text_input("AI에게 요청할 작업이나 질문을 입력하세요:", placeholder="예: 오늘 로봇 도구 테스트 일정 요약해줘")

if st.button("AI 요청 전송", type="primary"):
    if user_input:
        with st.spinner("구글 제미나이 AI가 답변을 생성하는 중..."):
            try:
                # 구글 GenAI 최신 라이브러리 구동
                client = genai.Client(api_key=gemini_key)
                response = client.models.generate_content(
                    model='gemini-2.5-flash',
                    contents=user_input,
                )
                
                # 결과 출력
                st.success("🤖 AI 답변 생성 완료!")
                st.write(response.text)
                
                # 3. 슬랙 알림 발송 기능
                slack_data = {"text": f"✅ [RKAI 알림]\n- 사용자 요청: {user_input}\n- AI가 답변을 성공적으로 제공했습니다."}
                requests.post(slack_url, json=slack_data)
                st.toast("🔔 슬랙(Slack) 채널로 연동 알림이 전송되었습니다!")

            except Exception as e:
                st.error(f"❌ 구동 중 에러 발생: {e}")
                # 에러 발생 시 슬랙으로 긴급 알림
                requests.post(slack_url, json={"text": f"🚨 [RKAI 에러 알림] 시스템 구동 중 에러 발생:\n{e}"})
    else:
        st.warning("내용을 입력한 뒤 전송 버튼을 눌러주세요.")

# 하단 정보 창
st.sidebar.markdown("### 🛠️ 시스템 상태")
st.sidebar.success("구글 AI 엔진: 연결 완료")
st.sidebar.success("슬랙 웹훅: 연결 완료")
google-genai
requests
