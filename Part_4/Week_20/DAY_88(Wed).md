## 프로젝트 진행사항

- sessionInformation.expireNow();
- sessionRegistry.removeSessionInformation(sessionInformation.getSessionId());
- 2개 동시에 사용 시 쿠키 세션이 사라지지 않아서 인증된 상태로 유지되며 그 상태로 정상 상태와 동일한 행위 가능
