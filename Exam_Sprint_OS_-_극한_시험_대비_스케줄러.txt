import React, { useState, useEffect, useRef, useMemo } from 'react';
import {
  Calendar, Clock, Flame, BookOpen, Cpu, Zap, Magnet, Layers,
  CheckCircle2, Circle, Play, Pause, RotateCcw, Coffee, Sparkles,
  AlertTriangle, ChevronRight, Plus, Trash2, Award, Brain,
  FileText, Upload, Send, Volume2, VolumeX, ShieldAlert,
  BarChart3, Target, Compass, Bookmark, HelpCircle, X, Check,
  Cloud, Key, Download, RefreshCw, Link2, CheckCheck, Copy, ExternalLink
} from 'lucide-react';
import { initializeApp } from 'firebase/app';
import {
  getAuth,
  signInWithCustomToken,
  signInAnonymously,
  onAuthStateChanged
} from 'firebase/auth';
import {
  getFirestore,
  doc,
  setDoc,
  getDoc,
  onSnapshot
} from 'firebase/firestore';

const appId = typeof __app_id !== 'undefined' ? __app_id : 'exam-sprint-os-2026';
let firebaseApp = null;
let auth = null;
let db = null;

try {
  if (typeof __firebase_config !== 'undefined' && __firebase_config) {
    const firebaseConfig = JSON.parse(__firebase_config);
    firebaseApp = initializeApp(firebaseConfig);
    auth = getAuth(firebaseApp);
    db = getFirestore(firebaseApp);
  }
} catch (err) {
  console.error('Cloud init fallback:', err);
}

const DEFAULT_OPENBOOK_INDEX = [
  { id: 1, chapter: '8단원', keyword: '캐리어 농도 / 페르미 준위 / 기본 수식', page: 'p. 8-12', note: '온도 의존성 그래프 주의' },
  { id: 2, chapter: '8단원', keyword: '표준 예제 8.4 (시험 빈출 변형)', page: 'p. 8-28', note: '도핑 농도 단위 cm⁻³ 확인' },
  { id: 3, chapter: '9단원', keyword: 'PN 접합 / 공핍층 폭 W 및 내장전위 Vbi', page: 'p. 9-05', note: '역방향 바이어스 인가 시 부호 주의' },
  { id: 4, chapter: '9단원', keyword: '소자 전류-전압(I-V) 특성 방정식', page: 'p. 9-19', note: '이상계수 및 포화전류 공식' }
];

const EXAMS = [
  {
    id: 'embedded',
    name: '임베디드 시스템 (ATmega128)',
    shortName: '임베디드(ATmega128)',
    date: '2026-10-21T09:00:00',
    endTime: '10:00',
    dayLabel: '10월 21일 (수) 09:00 - 10:00',
    difficulty: 'NORMAL',
    difficultyLabel: '할만함 (암기+응용)',
    type: '클로즈북 · 레지스터 & C코드 제어',
    color: 'cyan',
    accentHex: '#06B6D4',
    bgGlow: 'from-cyan-500/20 to-blue-600/5',
    borderGlow: 'border-cyan-500/30',
    textAccent: 'text-cyan-400',
    badgeBg: 'bg-cyan-500/15 text-cyan-300 border-cyan-500/30',
    strategy: '포트 제어(DDR/PORT/PIN), 외부 인터럽트, 타이머/카운터(CTC/PWM) 분주비 계산, UART/ADC 초기화 레지스터 비트 구성을 백지 복습하세요.'
  },
  {
    id: 'semicon',
    name: '전자반도체공학기초 (8·9단원)',
    shortName: '전자반도체(8·9장)',
    date: '2026-10-23T09:00:00',
    endTime: '10:30',
    dayLabel: '10월 23일 (금) 09:00 시작',
    difficulty: 'OPEN_BOOK',
    difficultyLabel: '오픈북 (속도전)',
    type: '오픈북 테스트 · 교재 8, 9단원',
    color: 'emerald',
    accentHex: '#10B981',
    bgGlow: 'from-emerald-500/20 to-teal-600/5',
    borderGlow: 'border-emerald-500/30',
    textAccent: 'text-emerald-400',
    badgeBg: 'bg-emerald-500/15 text-emerald-300 border-emerald-500/30',
    strategy: '오픈북은 "책을 안 보고도 어디에 무슨 공식과 예제가 있는지 3초 안에 찾는 사람"이 이깁니다. 8·9단원 포스트잇 인덱싱과 예제 풀이 흐름도를 미리 완성하세요.'
  },
  {
    id: 'circuit',
    name: '회로이론 (Circuit Theory)',
    shortName: '회로이론 [HELL]',
    date: '2026-10-26T09:00:00',
    endTime: '12:00',
    dayLabel: '10월 26일 (월) 09:00 - 12:00 (3시간)',
    difficulty: 'HELL',
    difficultyLabel: '난이도 HELL 🔥',
    type: '3시간 극한 계산 · 타임어택',
    color: 'rose',
    accentHex: '#F43F5E',
    bgGlow: 'from-rose-500/25 to-orange-600/10',
    borderGlow: 'border-rose-500/40',
    textAccent: 'text-rose-400',
    badgeBg: 'bg-rose-500/20 text-rose-300 border-rose-500/40',
    strategy: '금요일 오후부터 시작하면 절대 늦습니다. 월요일부터 매일 2~3시간씩 라플라스 변환, 2포트 회로망, 과도응답, 교류전력 킬러 문제를 꾸준히 풀어야 손이 굳지 않습니다.'
  },
  {
    id: 'em',
    name: '전자기학 (Electromagnetics)',
    shortName: '전자기학 [HELL]',
    date: '2026-10-26T12:00:00',
    endTime: '15:00',
    dayLabel: '10월 26일 (월) 12:00 - 15:00 (3시간 연강)',
    difficulty: 'HELL',
    difficultyLabel: '난이도 HELL 💀',
    type: '3시간 연강 · 수식 유도 & 공간적분',
    color: 'violet',
    accentHex: '#8B5CF6',
    bgGlow: 'from-violet-500/25 to-fuchsia-600/10',
    borderGlow: 'border-violet-500/40',
    textAccent: 'text-violet-400',
    badgeBg: 'bg-violet-500/20 text-violet-300 border-violet-500/40',
    strategy: '회로이론 직후 쉬는 시간 없이 12시~3시 연강입니다. 체력이 바닥난 상태에서 보므로 경계조건, 가우스/암페어 법칙 대칭성, 맥스웰 방정식 유도를 기계적으로 쓸 수 있어야 합니다.'
  }
];

const INITIAL_TOPICS = {
  embedded: [
    { id: 'emb-1', title: 'GPIO 포트 제어 (DDRx, PORTx, PINx 입출력 방향 및 풀업 설정)', tag: '필수암기', done: false },
    { id: 'emb-2', title: '외부 인터럽트 (EICRA, EICRB 트리거 조건, EIMSK, SREG I-bit)', tag: '빈출', done: false },
    { id: 'emb-3', title: 'Timer/Counter 0 & 1 (Normal, CTC, Fast PWM 모드 및 분주비 계산식)', tag: '킬러계산', done: false },
    { id: 'emb-4', title: 'UART 직렬 통신 (UBRR 보레이트 계산, UCSRnA/B/C 폴링 vs 인터럽트)', tag: '빈출', done: false },
    { id: 'emb-5', title: 'ADC 아날로그-디지털 변환 (ADMUX 기준전압/채널, ADCSRA 프리스케일러)', tag: '응용', done: false },
    { id: 'emb-6', title: '7-Segment (FND) 다이내믹 구동 및 채터링(Chattering) 방지 코드 구조', tag: '실전코드', done: false }
  ],
  semicon: [
    { id: 'semi-1', title: '8단원 핵심 공식 및 물성 파라미터 표 포스트잇 색인 완료', tag: '오픈북인덱스', done: false },
    { id: 'semi-2', title: '8단원 예제/연습문제 풀이 과정 형광펜 마킹 (조건 변형 대비)', tag: '오픈북필수', done: false },
    { id: 'semi-3', title: '9단원 핵심 소자 동작 영역별 전류-전압(I-V) 수식 페이지 번호 정리', tag: '오픈북인덱스', done: false },
    { id: 'semi-4', title: '9단원 에너지 밴드 다이어그램 & 캐리어 이동 메커니즘 개념 요약', tag: '서술형대비', done: false },
    { id: 'semi-5', title: '단위 환산 실수 방지 체크표 (eV, cm⁻³, C, 열전압 Vt=25.9mV 등)', tag: '실수방지', done: false }
  ],
  circuit: [
    { id: 'cir-1', title: '1차·2차 RLC 과도응답 (초기값/최종값, 고유주파수, 감쇠비 판별)', tag: 'HELL핵심', done: false },
    { id: 'cir-2', title: '페이저(Phasor) 변환 및 정현파 정상상태 교류전력 (유효/무효/역률보상)', tag: '빈출계산', done: false },
    { id: 'cir-3', title: '라플라스 변환 회로 해석 (s-영역 등가회로, 부분분수 전개, 전달함수 H(s))', tag: '킬러문항', done: false },
    { id: 'cir-4', title: '2포트(Two-Port) 회로망 파라미터 (Z, Y, h, ABCD 행렬 변환 및 직병렬 연결)', tag: 'HELL핵심', done: false },
    { id: 'cir-5', title: '주파수 응답, 공진 회로(Q-factor, 대역폭) 및 보데 선도(Bode Plot) 작도', tag: '고난도', done: false },
    { id: 'cir-6', title: '테브난/노턴 등가회로 (종속전원 포함 케이스) & 중첩의 원리 고속 풀이', tag: '기본기', done: false }
  ],
  em: [
    { id: 'em-1', title: '벡터 미적분 (직교/원통/구좌표계 Grad, Div, Curl 및 발산/스토크스 정리)', tag: '기초필수', done: false },
    { id: 'em-2', title: '정전계 (쿨롱 법칙, 가우스 법칙 대칭성 적분, 전위 V, 경계조건 유도)', tag: 'HELL핵심', done: false },
    { id: 'em-3', title: '포아송/라플라스 방정식 & 영상법(Method of Images) 전하 배치 해법', tag: '킬러문항', done: false },
    { id: 'em-4', title: '정자계 (비오-사바르 법칙, 암페어 주회법칙, 자계 벡터 포텐셜 A)', tag: 'HELL핵심', done: false },
    { id: 'em-5', title: '물질 내 자계 경계조건, 인덕턴스(내부/외부/상호) 및 자계 에너지 밀도', tag: '고난도', done: false },
    { id: 'em-6', title: '시변 전자계 (패러데이 전자기유도, 변위전류, 맥스웰 4대 방정식 적분/미분형)', tag: '만점방지용', done: false }
  ]
};

const WEEKLY_SCHEDULE = [
  {
    date: '2026-10-19',
    dayShort: '월',
    dayTitle: '10/19 (월) · D-2 임베디드 & 헬 과목 예열',
    subtitle: '수요일 임베디드 1회독 완성 + 회로/전자기학 감각 유지 (비율 5:5)',
    focusRatio: { embedded: 50, semicon: 0, circuit: 25, em: 25 },
    blocks: [
      { id: 'm1', time: '09:00 - 11:30', subject: 'embedded', title: 'ATmega128 구조 & GPIO / 외부 인터럽트 마스터', desc: 'DDRx, PORTx, PINx 비트 연산 및 EICRA/EIMSK 레지스터 암기 및 예제 코드 직접 손코딩' },
      { id: 'm2', time: '11:30 - 13:00', subject: 'rest', title: '점심 식사 및 뇌 피로 리셋 산책 (90분 보장)', desc: '햇빛 쬐며 가볍게 걷기, 스마트폰 쇼츠 시청 자제' },
      { id: 'm3', time: '13:00 - 15:30', subject: 'circuit', title: '[HELL 방어] 회로이론 1·2차 과도응답 & 페이저 핵심 문제', desc: '수요일 시험에만 몰두하면 회로이론 감각이 죽습니다. 대표 기출 8문항 자력 풀이' },
      { id: 'm4', time: '15:30 - 16:00', subject: 'rest', title: '오후 카페인/수분 보충 & 스트레칭 (30분)', desc: '눈 감고 15분 휴식 후 가벼운 간식' },
      { id: 'm5', time: '16:00 - 18:30', subject: 'embedded', title: 'ATmega128 타이머/카운터 (CTC, PWM) 분주비 계산 정복', desc: 'OCRn, TCCRn 레지스터 설정값 계산 공식 완벽 숙지' },
      { id: 'm6', time: '18:30 - 20:00', subject: 'rest', title: '저녁 식사 및 샤워 리프레시 (90분 보장)', desc: '충분한 영양 섭취 및 휴식' },
      { id: 'm7', time: '20:00 - 22:30', subject: 'em', title: '[HELL 방어] 전자기학 좌표계 벡터미적분 & 가우스/경계조건', desc: '원통/구좌표계 적분 요소(dl, dS, dV) 및 정전계 킬러 유형 6문제 풀이' },
      { id: 'm8', time: '22:30 - 23:30', subject: 'embedded', title: '임베디드 시스템 오늘 배운 레지스터 백지 복습', desc: '안 보고 레지스터 비트 이름 써보기 후 24:00 이전 취침' }
    ]
  },
  {
    date: '2026-10-20',
    dayShort: '화',
    dayTitle: '10/20 (화) · 임베디드 D-1 최종 점검 & 회로이론 킵고잉',
    subtitle: '내일 오전 9시 임베디드 시험 완벽 대비 + 회로이론 라플라스 방어',
    focusRatio: { embedded: 70, semicon: 0, circuit: 20, em: 10 },
    blocks: [
      { id: 'tu1', time: '09:00 - 11:30', subject: 'embedded', title: 'ATmega128 UART 통신 & ADC 변환 레지스터 총정리', desc: 'UBRR 보레이트 수식, ADMUX/ADCSRA 동작 순서 및 인터럽트 서비스 루틴(ISR) 코드 점검' },
      { id: 'tu2', time: '11:30 - 13:00', subject: 'rest', title: '점심 식사 & 낮잠 20분 (90분 보장)', desc: '내일 오전 9시 시험 컨디션을 위해 생체 리듬 맞추기' },
      { id: 'tu3', time: '13:00 - 15:00', subject: 'circuit', title: '[HELL 방어] 회로이론 라플라스 변환 회로해석 집중 훈련', desc: 's-영역 커패시터/인덕터 초기전압·전류 등가회로 변환 실수 방지 연습' },
      { id: 'tu4', time: '15:00 - 15:30', subject: 'rest', title: '브레이크 타임 (30분)', desc: '가벼운 스트레칭 및 환기' },
      { id: 'tu5', time: '15:30 - 18:30', subject: 'embedded', title: '임베디드 족보/과제 코드 전체 회독 & 예상 문제 시뮬레이션', desc: 'FND, 스위치 인터럽트, 타이머 조합 복합 응용코드 빈칸 채우기 및 전체 흐름 암기' },
      { id: 'tu6', time: '18:30 - 20:00', subject: 'rest', title: '저녁 식사 및 휴식 (90분 보장)', desc: '소화 잘 되는 메뉴 선택' },
      { id: 'tu7', time: '20:00 - 22:30', subject: 'embedded', title: 'ATmega128 전 범위 핵심 레지스터 치트시트 최종 암기', desc: '헷갈리는 레지스터 비트 위치(EICRA, TCCR0, UCSRB 등) 최종 각인' },
      { id: 'tu8', time: '22:30 - 23:15', subject: 'em', title: '전자기학 핵심 공식 노션/노트 가볍게 1회독 (45분)', desc: '수식 감각만 유지하고 내일 9시 시험을 위해 23:30 취침!' }
    ]
  },
  {
    date: '2026-10-21',
    dayShort: '수',
    dayTitle: '10/21 (수) · [시험 1] 임베디드 격파 & 오픈북/헬 과목 전환',
    subtitle: '09:00-10:00 임베디드 시험 완료 후 보상 휴식 → 오후 반도체 & 전자기학',
    focusRatio: { embedded: 20, semicon: 35, circuit: 15, em: 30 },
    blocks: [
      { id: 'w1', time: '07:30 - 08:50', subject: 'embedded', title: '기상 및 시험장 이동 · 레지스터 요약본 최종 스캔', desc: '분주비 공식, 인터럽트 벡터 번호, 입출력 방향 레지스터 실수 포인트 점검' },
      { id: 'w2', time: '09:00 - 10:00', subject: 'embedded', title: '🎯 [실전 시험 1] 임베디드 시스템 (ATmega128)', desc: '09:00 - 10:00 (60분) · 침착하게 레지스터 초기화 조건과 주석까지 깔끔하게 작성!' },
      { id: 'w3', time: '10:00 - 13:30', subject: 'rest', title: '🎉 첫 시험 종료! 보상 휴식 & 맛있는 점심 + 낮잠 (3시간 30분 보장)', desc: '첫 시험의 긴장을 완전히 풀고 에너지를 재충전하세요. (절대 자책하거나 피곤한 상태로 바로 책상에 앉지 말 것)' },
      { id: 'w4', time: '13:30 - 16:30', subject: 'semicon', title: '전자반도체공학기초 8단원 정독 & 오픈북 포스트잇 인덱싱', desc: '금요일 오픈북 시험 대비: 8단원 모든 공식, 그래프, 예제 옆에 색깔별 인덱스 탭 부착' },
      { id: 'w5', time: '16:30 - 17:00', subject: 'rest', title: '오후 휴식 (30분)', desc: '커피/차 한 잔과 스트레칭' },
      { id: 'w6', time: '17:00 - 19:00', subject: 'em', title: '[HELL 집중] 전자기학 포아송/라플라스 & 영상법 심화 풀이', desc: '가장 까다로운 정전계 경계치 문제와 영상전하 배치 유형 완전 정복' },
      { id: 'w7', time: '19:00 - 20:30', subject: 'rest', title: '저녁 식사 및 휴식 (90분 보장)', desc: '편안한 음악 감상 및 산책' },
      { id: 'w8', time: '20:30 - 23:00', subject: 'circuit', title: '[HELL 집중] 회로이론 2포트 회로망 (Z, Y, h, ABCD) 마스터', desc: '행렬 변환 공식 유도와 2포트 직렬/병렬/종속 연결 기출문제 풀이' }
    ]
  },
  {
    date: '2026-10-22',
    dayShort: '목',
    dayTitle: '10/22 (목) · 반도체 오픈북 단권화 완성 & 헬 과목 문제풀이',
    subtitle: '내일 오전 9시 전자반도체 오픈북 완벽 세팅 + 회로/전자기학 실전 훈련',
    focusRatio: { embedded: 0, semicon: 55, circuit: 25, em: 20 },
    blocks: [
      { id: 'th1', time: '09:00 - 12:00', subject: 'semicon', title: '전자반도체공학기초 9단원 정독 & 오픈북 인덱스 완성', desc: '9단원 핵심 소자 수식, 에너지 밴드 다이어그램, 예제 풀이 순서를 A4 1장 목차표(Quick Finder)로 제작' },
      { id: 'th2', time: '12:00 - 13:30', subject: 'rest', title: '점심 식사 및 휴식 (90분 보장)', desc: '여유롭게 식사 후 리프레시' },
      { id: 'th3', time: '13:30 - 16:00', subject: 'semicon', title: '전자반도체 8·9단원 연습문제 실전 오픈북 속도전 리허설', desc: '실제 책만 보고 문제를 풀 때 어디서 시간이 걸리는지 체크하고 계산기 수식 입력 연습' },
      { id: 'th4', time: '16:00 - 16:30', subject: 'rest', title: '오후 브레이크 (30분)', desc: '뇌 휴식 및 간식' },
      { id: 'th5', time: '16:30 - 19:00', subject: 'em', title: '[HELL 집중] 전자기학 정자계 (비오-사바르, 암페어, 인덕턴스)', desc: '자계 세기 H 계산, 자속밀도 B, 동축케이블/솔레노이드 인덕턴스 유도 완벽 정리' },
      { id: 'th6', time: '19:00 - 20:30', subject: 'rest', title: '저녁 식사 및 휴식 (90분 보장)', desc: '체력 비축' },
      { id: 'th7', time: '20:30 - 22:30', subject: 'circuit', title: '[HELL 집중] 회로이론 주파수 응답·공진회로·보데선도', desc: '직렬/병렬 RLC 공진 Q-factor 및 전달함수 폴/제로 분석' },
      { id: 'th8', time: '22:30 - 23:15', subject: 'semicon', title: '내일 오픈북 시험 지참물(교재, 인덱스표, 공학용계산기) 점검', desc: '오전 9시 시험을 위해 23:30 취침' }
    ]
  },
  {
    date: '2026-10-23',
    dayShort: '금',
    dayTitle: '10/23 (금) · [시험 2] 반도체 오픈북 격파 → 72시간 HELL 돌입',
    subtitle: '오전 오픈북 시험 종료 후 휴식 → 금요일 오후부터 회로+전자기학 올인!',
    focusRatio: { embedded: 0, semicon: 20, circuit: 40, em: 40 },
    blocks: [
      { id: 'f1', time: '07:30 - 08:50', subject: 'semicon', title: '기상 및 8·9단원 페이지 인덱스 맵 최종 점검', desc: '어떤 키워드가 나오면 몇 페이지를 펼칠지 시뮬레이션' },
      { id: 'f2', time: '09:00 - 10:30', subject: 'semicon', title: '🎯 [실전 시험 2] 전자반도체공학기초 (8·9단원 오픈북)', desc: '09:00 시작 · 오픈북의 핵심은 단위(cm⁻³, eV) 통일과 빠른 예제 매칭입니다!' },
      { id: 'f3', time: '10:30 - 14:00', subject: 'rest', title: '🎉 금요일 시험 종료! 꿀휴식 & 점심 특식 + 낮잠 (3시간 30분 보장)', desc: '이제 월요일까지 시험이 없습니다. 푹 쉬고 오후 2시부터 진짜 전쟁(회로+전자기학)을 시작합니다.' },
      { id: 'f4', time: '14:00 - 17:00', subject: 'circuit', title: '🔥 [HELL 올인 1] 회로이론 전 범위 개념 빈틈 색출 & 고난도 문제', desc: '과도응답 + 교류정상상태 + 라플라스 변환 킬러 문제 15선 집중 격파' },
      { id: 'f5', time: '17:00 - 17:30', subject: 'rest', title: '오후 휴식 및 스트레칭 (30분)', desc: '당 보충 및 승모근 스트레칭' },
      { id: 'f6', time: '17:30 - 19:00', subject: 'em', title: '🔥 [HELL 올인 2] 전자기학 시변전자계 & 맥스웰 방정식 유도', desc: '패러데이 법칙, 변위전류 밀도 Jd 유도 및 미분형/적분형 변환 암기' },
      { id: 'f7', time: '19:00 - 20:30', subject: 'rest', title: '저녁 식사 및 자유 휴식 (90분 보장)', desc: '금요일 저녁, 좋아하는 영상 1편 보며 식사' },
      { id: 'f8', time: '20:30 - 23:30', subject: 'em', title: '🔥 [HELL 올인 3] 전자기학 전 범위 기출/연습문제 1차 총정리', desc: '안 풀리는 문제 별표 쳐두고 풀이 논리 단계별 구조화' }
    ]
  },
  {
    date: '2026-10-24',
    dayShort: '토',
    dayTitle: '10/24 (토) · [HELL DAY 1] 회로이론 & 전자기학 6시간 연강 체력 훈련',
    subtitle: '월요일 실제 시험 시간표(09~12시 회로 / 12~15시 전자기학)와 동일한 바이오리듬 훈련!',
    focusRatio: { embedded: 0, semicon: 0, circuit: 50, em: 50 },
    blocks: [
      { id: 'sa1', time: '09:00 - 12:00', subject: 'circuit', title: '⚡ [실전 시간 동기화] 회로이론 3시간 연속 실전 문제풀이', desc: '월요일 09:00~12:00 본시험과 똑같은 시간에 타이머 맞춰두고 최고난도 문제지 풀기' },
      { id: 'sa2', time: '12:00 - 13:30', subject: 'rest', title: '점심 식사 및 뇌 산소 공급 산책 (90분 보장)', desc: '오전 3시간 집중 후 완벽한 휴식' },
      { id: 'sa3', time: '13:30 - 16:30', subject: 'em', title: '⚡ [실전 시간 동기화] 전자기학 3시간 연속 증명 & 계산 훈련', desc: '정전계·정자계·시변계 핵심 증명 5개 백지 서술 + 복합 응용문제 풀이' },
      { id: 'sa4', time: '16:30 - 17:30', subject: 'rest', title: '오후 파워 리커버리 휴식 (60분 보장)', desc: '고강도 6시간 공부 후 낮잠 30분 또는 가벼운 샤워' },
      { id: 'sa5', time: '17:30 - 19:30', subject: 'circuit', title: '회로이론 오답 노트 & 계산 실수 다발 구간 집중 교정', desc: '복소수 연산(크기/위상), 부분분수 계수 구하기, 행렬 역행렬 검산법 훈련' },
      { id: 'sa6', time: '19:30 - 21:00', subject: 'rest', title: '저녁 식사 및 휴식 (90분 보장)', desc: '든든한 식사로 주말 체력 유지' },
      { id: 'sa7', time: '21:00 - 23:30', subject: 'em', title: '전자기학 오답 노트 & 좌표계 적분 공식 완벽 체화', desc: '경계조건 수식 부호 실수, 구좌표계 적분 범위 실수 집중 교정' }
    ]
  },
  {
    date: '2026-10-25',
    dayShort: '일',
    dayTitle: '10/25 (일) · [HELL DAY 2] 결전의 전날 · 최종 단권화 & 컨디션 피크',
    subtitle: '내일 6시간 연속 지옥의 마라톤 시험! 새로운 문제보다 오답/핵심공식 완벽 각인',
    focusRatio: { embedded: 0, semicon: 0, circuit: 50, em: 50 },
    blocks: [
      { id: 'su1', time: '09:00 - 12:00', subject: 'circuit', title: '🔥 회로이론 전 범위 최종 모의고사 & 핵심 풀이 루틴 총정리', desc: '문제를 보자마자 어떤 해석법(마디/망로/라플라스/페이저)을 쓸지 10초 안에 판단하는 훈련' },
      { id: 'su2', time: '12:00 - 13:30', subject: 'rest', title: '점심 식사 및 휴식 (90분 보장)', desc: '과식 금지, 산뜻한 식사' },
      { id: 'su3', time: '13:30 - 16:30', subject: 'em', title: '🔥 전자기학 전 범위 최종 모의고사 & 유도 과정 백지 총정리', desc: '가우스/스토크스 정리부터 맥스웰 방정식까지 막히는 유도 과정 제로화' },
      { id: 'su4', time: '16:30 - 17:15', subject: 'rest', title: '오후 휴식 & 내일 간식(초콜릿/에너지바/포도당캔디) 준비 (45분)', desc: '내일 12시 회로이론이 끝나자마자 전자기학이 시작되므로 쉬는 시간에 바로 먹을 당분 필수 준비!' },
      { id: 'su5', time: '17:15 - 19:15', subject: 'circuit', title: '회로이론 A4 1장 극약처방 요약본(Failsafe Sheet) 암기', desc: '라플라스 변환쌍, 2포트 파라미터 변환표, 2차 회로 공식 최종 점검' },
      { id: 'su6', time: '19:15 - 20:30', subject: 'rest', title: '저녁 식사 및 따뜻한 샤워 (75분 보장)', desc: '긴장 완화' },
      { id: 'su7', time: '20:30 - 22:30', subject: 'em', title: '전자기학 A4 1장 극약처방 요약본(Failsafe Sheet) 암기', desc: '벡터 항등식, 경계조건, 정전용량/인덕턴스 공식 최종 스캔 후 23:00 무조건 취침! (6시간 연강은 수면이 생명)' }
    ]
  },
  {
    date: '2026-10-26',
    dayShort: '월(결전)',
    dayTitle: '10/26 (월) · ⚔️ D-DAY 지옥의 6시간 연속 시험 (회로이론 + 전자기학)',
    subtitle: '09:00~12:00 회로이론 → 12:00~15:00 전자기학 · 초인적인 집중력으로 피날레!',
    focusRatio: { embedded: 0, semicon: 0, circuit: 50, em: 50 },
    blocks: [
      { id: 'd1', time: '07:00 - 08:45', subject: 'circuit', title: '기상 · 아침 식사 · 회로이론/전자기학 A4 요약본 최종 스캔', desc: '초콜릿/에너지음료/물 챙겨서 시험장 입실 완료' },
      { id: 'd2', time: '09:00 - 12:00', subject: 'circuit', title: '⚔️ [최종 보스 1] 회로이론 본시험 (180분)', desc: '아는 문제부터 정확히 풀고, 검산(초기값/최종값 정리 활용)까지 완벽하게!' },
      { id: 'd3', time: '12:00 - 15:00', subject: 'em', title: '⚔️ [최종 보스 2] 전자기학 본시험 (180분 연강)', desc: '시작 전 포도당/초콜릿 섭취! 대칭성 조건 명시하고 단위벡터 방향까지 정확하게 서술!' },
      { id: 'd4', time: '15:00 ~', subject: 'rest', title: '🏆 모든 시험 종료! 완벽한 해방 & 자유 시간', desc: '일주일간의 극한 스프린트 완주! 수고하셨습니다!' }
    ]
  }
];

const CHEAT_SHEETS = {
  embedded: [
    { reg: 'DDRx / PORTx / PINx', desc: 'DDRx=1(출력)/0(입력), 입력일 때 PORTx=1이면 내부 풀업 저항 활성화, 입력값 읽기는 PINx' },
    { reg: 'EICRA / EICRB / EIMSK', desc: 'INT0~3은 EICRA, INT4~7은 EICRB (ISCn1, ISCn0: 00=Low, 10=하강엣지, 11=상승엣지)' },
    { reg: 'TCCR0 (Timer 0)', desc: 'WGM01:00 (00=Normal, 10=CTC, 11=Fast PWM) / CS02:00 분주비 설정 / TCNT0 카운터, OCR0 비교값' },
    { reg: '타이머 주기 공식', desc: 'f_timer = f_cpu / Prescaler, 오버플로우 주기 T = (1 / f_timer) × (256 - TCNT0) 또는 (OCR0 + 1)' },
    { reg: 'UART (UCSRnA/B/C, UBRRn)', desc: 'RXEN/TXEN 활성화, UDRE(송신버퍼 비어있음 대기), UBRR = (f_cpu / (16 × Baud)) - 1' },
    { reg: 'ADC (ADMUX, ADCSRA)', desc: 'REFS1:0 기준전압, MUX 채널선택 / ADEN=1, ADSC=1(변환시작), ADIF=1(완료 플래그), 10비트 결과(ADCL 먼저 읽고 ADCH)' }
  ],
  semicon: [
    { item: '색인 탭 규칙', detail: '파란색=물성상수/표, 노란색=핵심공식(8·9장), 빨간색=수업시간 강조 예제' },
    { item: '자주 쓰는 상수', detail: 'q = 1.6×10⁻¹⁹ C, kT/q ≈ 0.0259 V (300K), ε₀ = 8.854×10⁻¹⁴ F/cm, ε_si = 11.7 ε₀' },
    { item: '오픈북 시간 배분', detail: '문제 읽고 30초 내 해당 단원 예제 페이지 오픈 → 변수 대입 전 단위(cm vs m) 무조건 확인' }
  ]
};

const FLASHCARDS = {
  embedded: [
    { q: 'ATmega128에서 PORTA의 0~3번 핀은 입력(내부 풀업 활성화), 4~7번 핀은 출력으로 설정하는 C코드는?', a: 'DDRA = 0xF0; (상위 4비트 출력=1, 하위 4비트 입력=0)\nPORTA = 0x0F; (입력 핀에 1을 써서 내부 풀업 저항 ON)' },
    { q: '시스템 클럭 16MHz, 분주비 64, CTC 모드에서 1ms(1kHz) 주기로 비교일치 인터럽트를 발생시키려면 OCR0 값은?', a: 'f_timer = 16MHz / 64 = 250kHz (1틱당 4μs)\n1ms / 4μs = 250개 카운트 필요 → OCR0 = 250 - 1 = 249' },
    { q: '외부 인터럽트 INT0을 하강 엣지(Falling Edge) 트리거로 설정하고 허용하는 레지스터 코드는?', a: 'EICRA |= (1 << ISC01); EICRA &= ~(1 << ISC00); (ISC01:00 = 10)\nEIMSK |= (1 << INT0);\nsei(); (전체 인터럽트 허용)' }
  ],
  semicon: [
    { q: '[오픈북 3초 체크] 300K 실리콘에서 열전압(Thermal Voltage) V_T 값과 진성 캐리어 농도 n_i의 대표값은?', a: 'V_T = kT/q ≈ 0.0259 V (25.9 mV)\n실리콘(Si) n_i ≈ 1.5 × 10¹⁰ cm⁻³ (교재 상수표 페이지 확인 필수!)' },
    { q: '[오픈북 함정 방어] 공핍층 폭 W이나 정전용량 C 계산 시 가장 많이 틀리는 단위 환산 포인트는?', a: '유전율 ε₀ = 8.854 × 10⁻¹⁴ F/cm 를 쓸 때는 도핑 농도 N_A, N_D도 반드시 cm⁻³ 단위로 통일해야 함! (m⁻³와 혼용 절대 금지)' }
  ],
  circuit: [
    { q: '[회로이론 HELL] 초기 전압 V₀가 충전된 커패시터 C를 라플라스(s-영역) 직렬 등가회로로 변환하면?', a: '임피던스 1/(sC) 와 직렬로 전압원 V₀/s 가 연결됨 (극성: 초기 전압 방향 유지)\n병렬(노턴) 변환 시에는 전류원 C·V₀ 가 병렬 연결됨' },
    { q: '[회로이론 HELL] 2포트 회로망에서 대칭성(Symmetry)과 상반성(Reciprocity) 조건을 Z파라미터와 ABCD파라미터로 쓰면?', a: '• Z 파라미터: 대칭 Z₁₁ = Z₂₂ / 상반 Z₁₂ = Z₂₁\n• ABCD 파라미터: 대칭 A = D / 상반 AD - BC = 1' },
    { q: '[회로이론 HELL] 라플라스 초기값 정리와 최종값 정리 공식 및 사용 조건은?', a: '• 초기값: f(0⁺) = lim(s→∞) sF(s)\n• 최종값: f(∞) = lim(s→0) sF(s) (단, sF(s)의 모든 극점이 s-평면 좌반면에 있을 때만 성립)' }
  ],
  em: [
    { q: '[전자기학 HELL] 유전체 경계면(ε₁, ε₂)에서 전계 E와 전속밀도 D의 경계조건 2가지는?', a: '1) 접선 성분 연속: E₁t = E₂t (즉, D₁t / ε₁ = D₂t / ε₂)\n2) 법선 성분 불연속: D₁n - D₂n = ρ_s (자유 표면전하밀도가 0이면 D₁n = D₂n)' },
    { q: '[전자기학 HELL] 맥스웰 4대 방정식 미분형을 순서대로 쓰시오.', a: '1) ∇ · D = ρ_v (가우스 정전계 법칙)\n2) ∇ · B = 0 (자속 보존의 법칙 / 자하 없음)\n3) ∇ × E = -∂B/∂t (패러데이 전자기유도 법칙)\n4) ∇ × H = J + ∂D/∂t (암페어-맥스웰 주회법칙)' },
    { q: '[전자기학 HELL] 자성체 경계면(μ₁, μ₂)에서 자계 H와 자속밀도 B의 경계조건은?', a: '1) 법선 성분 연속: B₁n = B₂n\n2) 접선 성분: H₁t - H₂t = K (표면전류밀도 K=0이면 H₁t = H₂t)' }
  ]
};

const getBlockDurationHours = (timeStr) => {
  try {
    if (timeStr.includes('~')) return 2;
    const [start, end] = timeStr.split('-').map((s) => s.trim());
    const [sh, sm] = start.split(':').map(Number);
    const [eh, em] = end.split(':').map(Number);
    return Math.max(0.5, (eh + em / 60) - (sh + sm / 60));
  } catch {
    return 1.5;
  }
};

export default function ExamSprintOS() {
  const [activeTab, setActiveTab] = useState('dashboard');
  const [selectedDayIdx, setSelectedDayIdx] = useState(0);
  const [scheduleViewMode, setScheduleViewMode] = useState('calendar');
  const [calendarFilter, setCalendarFilter] = useState('all');
  const [selectedCalBlock, setSelectedCalBlock] = useState(null);
  const [activeVaultSubject, setActiveVaultSubject] = useState('circuit');
  const [flippedCards, setFlippedCards] = useState({});

  const [completedBlocks, setCompletedBlocks] = useState({ m1: false });
  const [topics, setTopics] = useState(INITIAL_TOPICS);
  const [openBookIndex, setOpenBookIndex] = useState(DEFAULT_OPENBOOK_INDEX);

  const [user, setUser] = useState(null);
  const [cloudStatus, setCloudStatus] = useState('connecting');
  const [lastSavedAt, setLastSavedAt] = useState(null);
  const [syncRoomInput, setSyncRoomInput] = useState('');
  const [activeSyncRoom, setActiveSyncRoom] = useState('');
  const [customApiKey, setCustomApiKey] = useState('');
  const [showCloudModal, setShowCloudModal] = useState(false);
  const isRemoteUpdateRef = useRef(false);
  const initialLoadCompleteRef = useRef(false);

  const [newTopicText, setNewTopicText] = useState('');
  const [newTopicTag, setNewTopicTag] = useState('핵심');
  const [newObChapter, setNewObChapter] = useState('8단원');
  const [newObKeyword, setNewObKeyword] = useState('');
  const [newObPage, setNewObPage] = useState('');
  const [newObNote, setNewObNote] = useState('');
  const [toastMsg, setToastMsg] = useState(null);
  const [cheatSheetModal, setCheatSheetModal] = useState(null);

  const [timerMode, setTimerMode] = useState('deep');
  const [totalDuration, setTotalDuration] = useState(90 * 60);
  const [timeLeft, setTimeLeft] = useState(90 * 60);
  const [isRunning, setIsRunning] = useState(false);
  const [soundEnabled, setSoundEnabled] = useState(true);
  const [isStandbyMode, setIsStandbyMode] = useState(false);

  const [aiSubject, setAiSubject] = useState('circuit');
  const [aiPrompt, setAiPrompt] = useState('');
  const [aiImageBase64, setAiImageBase64] = useState(null);
  const [aiImageMime, setAiImageMime] = useState('image/png');
  const [aiImagePreview, setAiImagePreview] = useState(null);
  const [aiResponse, setAiResponse] = useState('');
  const [aiLoading, setAiLoading] = useState(false);

  const showToast = (msg) => {
    setToastMsg(msg);
    setTimeout(() => setToastMsg(null), 3200);
  };

  useEffect(() => {
    if (!auth) {
      setCloudStatus('offline');
      return;
    }
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (err) {
        console.error('Auth error:', err);
        setCloudStatus('offline');
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
      if (!currentUser) {
        setCloudStatus('offline');
      }
    });
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user || !db) return;

    const docRef = activeSyncRoom
      ? doc(db, 'artifacts', appId, 'public', 'data', 'syncRooms', activeSyncRoom)
      : doc(db, 'artifacts', appId, 'users', user.uid, 'sprintData', 'mainState');

    setCloudStatus('connecting');

    const unsubscribe = onSnapshot(
      docRef,
      (snapshot) => {
        if (snapshot.exists()) {
          const data = snapshot.data();
          isRemoteUpdateRef.current = true;
          if (data.completedBlocks) setCompletedBlocks(data.completedBlocks);
          if (data.topics) setTopics(data.topics);
          if (data.openBookIndex) setOpenBookIndex(data.openBookIndex);
          if (typeof data.customApiKey === 'string' && !activeSyncRoom) {
            setCustomApiKey(data.customApiKey);
          }
          if (data.updatedAt) setLastSavedAt(data.updatedAt);
          setTimeout(() => {
            isRemoteUpdateRef.current = false;
            initialLoadCompleteRef.current = true;
          }, 120);
        } else {
          initialLoadCompleteRef.current = true;
        }
        setCloudStatus('synced');
      },
      (err) => {
        console.error('Snapshot listener error:', err);
        setCloudStatus('offline');
      }
    );

    return () => unsubscribe();
  }, [user, activeSyncRoom]);

  useEffect(() => {
    if (!user || !db || !initialLoadCompleteRef.current || isRemoteUpdateRef.current) return;

    const saveTimer = setTimeout(async () => {
      try {
        setCloudStatus('saving');
        const timestamp = new Date().toLocaleTimeString('ko-KR', {
          hour: '2-digit',
          minute: '2-digit',
          second: '2-digit'
        });

        const payload = {
          completedBlocks,
          topics,
          openBookIndex,
          updatedAt: timestamp,
          ownerUid: user.uid
        };

        if (!activeSyncRoom) {
          payload.customApiKey = customApiKey;
        }

        const docRef = activeSyncRoom
          ? doc(db, 'artifacts', appId, 'public', 'data', 'syncRooms', activeSyncRoom)
          : doc(db, 'artifacts', appId, 'users', user.uid, 'sprintData', 'mainState');

        await setDoc(docRef, payload, { merge: true });
        setLastSavedAt(timestamp);
        setCloudStatus('synced');
      } catch (err) {
        console.error('Auto-save error:', err);
        setCloudStatus('offline');
      }
    }, 500);

    return () => clearTimeout(saveTimer);
  }, [completedBlocks, topics, openBookIndex, customApiKey, user, activeSyncRoom]);

  const exportBackupJson = () => {
    const backupData = {
      version: 'exam-sprint-os-v2',
      exportedAt: new Date().toISOString(),
      completedBlocks,
      topics,
      openBookIndex
    };
    const blob = new Blob([JSON.stringify(backupData, null, 2)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `exam_sprint_backup_${new Date().toISOString().slice(0, 10)}.json`;
    a.click();
    URL.revokeObjectURL(url);
    showToast('공부 진행도 백업 파일(.json)이 다운로드되었습니다!');
  };

  const importBackupJson = (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = () => {
      try {
        const parsed = JSON.parse(reader.result);
        if (parsed.completedBlocks) setCompletedBlocks(parsed.completedBlocks);
        if (parsed.topics) setTopics(parsed.topics);
        if (parsed.openBookIndex) setOpenBookIndex(parsed.openBookIndex);
        showToast('백업 파일에서 학습 진도와 체크리스트를 성공적으로 복원했습니다!');
      } catch (err) {
        showToast('올바른 JSON 백업 파일이 아닙니다.');
      }
    };
    reader.readAsText(file);
  };

  const handleConnectSyncRoom = (e) => {
    e.preventDefault();
    const cleaned = syncRoomInput.trim().toUpperCase().replace(/[^A-Z0-9_-]/g, '');
    if (cleaned.length < 3) {
      showToast('동기화 코드는 영문/숫자 3글자 이상으로 입력해주세요 (예: PASS1026)');
      return;
    }
    setActiveSyncRoom(cleaned);
    showToast(`동기화 코드 [${cleaned}]에 연결되었습니다! 다른 기기에서도 같은 코드를 입력하세요.`);
  };

  const playChime = () => {
    if (!soundEnabled) return;
    try {
      const ctx = new (window.AudioContext || window.webkitAudioContext)();
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(523.25, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(1046.5, ctx.currentTime + 0.35);
      gain.gain.setValueAtTime(0.15, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.6);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.6);
    } catch (e) {
      // Ignore audio context errors
    }
  };

  useEffect(() => {
    let interval = null;
    if (isRunning && timeLeft > 0) {
      interval = setInterval(() => {
        setTimeLeft((prev) => prev - 1);
      }, 1000);
    } else if (isRunning && timeLeft === 0) {
      setIsRunning(false);
      playChime();
      showToast('타이머 세션이 완료되었습니다! 계획된 휴식을 취하거나 다음 블록으로 이동하세요.');
    }
    return () => clearInterval(interval);
  }, [isRunning, timeLeft]);

  const selectTimerPreset = (mode) => {
    setIsRunning(false);
    setTimerMode(mode);
    const durations = {
      focus50: 50 * 60,
      deep: 90 * 60,
      hell: 180 * 60,
      rest: 15 * 60
    };
    const nextSec = durations[mode] || 90 * 60;
    setTotalDuration(nextSec);
    setTimeLeft(nextSec);
  };

  const adjustTimerMinutes = (deltaMins) => {
    const deltaSec = deltaMins * 60;
    setTimeLeft((prev) => {
      const updated = Math.max(60, prev + deltaSec);
      setTotalDuration((currTotal) => Math.max(currTotal, updated));
      return updated;
    });
  };

  const formatTimer = (sec) => {
    const hrs = Math.floor(sec / 3600);
    const mins = Math.floor((sec % 3600) / 60);
    const secs = sec % 60;
    if (hrs > 0) {
      return `${String(hrs).padStart(2, '0')}:${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
    }
    return `${String(mins).padStart(2, '0')}:${String(secs).padStart(2, '0')}`;
  };

  const timerProgressPercent = useMemo(() => {
    if (!totalDuration || totalDuration <= 0) return 0;
    const elapsed = Math.max(0, totalDuration - timeLeft);
    return Math.min(100, Math.round((elapsed / totalDuration) * 100));
  }, [timeLeft, totalDuration]);

  const estimatedEndTimeString = useMemo(() => {
    const finishDate = new Date(Date.now() + timeLeft * 1000);
    return finishDate.toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' });
  }, [timeLeft]);

  const timerTheme = useMemo(() => {
    if (timerMode === 'rest') {
      return {
        name: '뇌 피로 리커버리 휴식',
        subtitle: '스마트폰을 내려놓고 눈을 감거나 가볍게 스트레칭하세요',
        stroke: '#34D399',
        glow: 'shadow-emerald-500/25',
        badge: 'bg-emerald-500/15 text-emerald-300 border-emerald-500/30',
        button: 'bg-emerald-500 hover:bg-emerald-400 text-zinc-950'
      };
    }
    if (timerMode === 'hell') {
      return {
        name: '회로·전자기학 180분 실전 내구레이스',
        subtitle: '10/26(월) 6시간 연강 시험 대비 극한 타임어택 모드',
        stroke: '#FB7185',
        glow: 'shadow-rose-500/25',
        badge: 'bg-rose-500/15 text-rose-300 border-rose-500/30',
        button: 'bg-rose-500 hover:bg-rose-400 text-white'
      };
    }
    if (timerMode === 'deep') {
      return {
        name: '킬러문항 90분 딥워크 세션',
        subtitle: '고난도 수식 유도 및 기출문제 자력 풀이 집중 시간',
        stroke: '#A78BFA',
        glow: 'shadow-violet-500/25',
        badge: 'bg-violet-500/15 text-violet-300 border-violet-500/30',
        button: 'bg-violet-500 hover:bg-violet-400 text-white'
      };
    }
    return {
      name: '표준 50분 초집중 스프린트',
      subtitle: '핵심 개념 암기 및 레지스터·공식 백지 복습 세션',
      stroke: '#38BDF8',
      glow: 'shadow-sky-500/25',
      badge: 'bg-sky-500/15 text-sky-300 border-sky-500/30',
      button: 'bg-sky-500 hover:bg-sky-400 text-zinc-950'
    };
  }, [timerMode]);

  const toggleBlock = (blockId) => {
    setCompletedBlocks((prev) => ({
      ...prev,
      [blockId]: !prev[blockId]
    }));
  };

  const toggleTopic = (subjId, topicId) => {
    setTopics((prev) => ({
      ...prev,
      [subjId]: prev[subjId].map((item) =>
        item.id === topicId ? { ...item, done: !item.done } : item
      )
    }));
  };

  const progressMetrics = useMemo(() => {
    let totalAllBlocks = 0;
    let completedAllBlocks = 0;
    const subjectProgress = {};
    const subjectHours = { embedded: 0, semicon: 0, circuit: 0, em: 0, rest: 0 };

    WEEKLY_SCHEDULE.forEach((day) => {
      day.blocks.forEach((b) => {
        totalAllBlocks++;
        if (completedBlocks[b.id]) completedAllBlocks++;
        const hrs = getBlockDurationHours(b.time);
        if (subjectHours[b.subject] !== undefined) {
          subjectHours[b.subject] += hrs;
        }
      });
    });

    ['embedded', 'semicon', 'circuit', 'em'].forEach((subj) => {
      const list = topics[subj] || [];
      if (list.length === 0) {
        subjectProgress[subj] = 0;
      } else {
        const done = list.filter((i) => i.done).length;
        subjectProgress[subj] = Math.round((done / list.length) * 100);
      }
    });

    const overallPct = totalAllBlocks > 0 ? Math.round((completedAllBlocks / totalAllBlocks) * 100) : 0;
    return { overallPct, subjectProgress, subjectHours };
  }, [completedBlocks, topics]);

  const getDiffString = (dateStr) => {
    const diff = new Date(dateStr) - new Date('2026-10-18T21:10:00');
    const days = Math.floor(diff / (1000 * 60 * 60 * 24));
    if (days === 0) return 'D-DAY (오늘)';
    if (days > 0) return `D-${days}`;
    return '시험종료';
  };

  const addCustomTopic = (e) => {
    e.preventDefault();
    if (!newTopicText.trim()) return;
    const newItem = {
      id: `custom-${Date.now()}`,
      title: newTopicText.trim(),
      tag: newTopicTag,
      done: false
    };
    setTopics((prev) => ({
      ...prev,
      [activeVaultSubject]: [...prev[activeVaultSubject], newItem]
    }));
    setNewTopicText('');
    showToast('새로운 시험 범위 체크리스트 항목이 추가되었습니다.');
  };

  const deleteTopic = (subjId, topicId) => {
    setTopics((prev) => ({
      ...prev,
      [subjId]: prev[subjId].filter((item) => item.id !== topicId)
    }));
  };

  const addOpenBookItem = (e) => {
    e.preventDefault();
    if (!newObKeyword.trim() || !newObPage.trim()) return;
    setOpenBookIndex((prev) => [
      ...prev,
      {
        id: Date.now(),
        chapter: newObChapter,
        keyword: newObKeyword.trim(),
        page: newObPage.trim(),
        note: newObNote.trim() || '-'
      }
    ]);
    setNewObKeyword('');
    setNewObPage('');
    setNewObNote('');
    showToast('오픈북 빠른 색인(Quick Finder)에 등록되었습니다.');
  };

  const removeOpenBookItem = (id) => {
    setOpenBookIndex((prev) => prev.filter((item) => item.id !== id));
  };

  const handleFileUpload = (e) => {
    const file = e.target.files?.[0];
    if (!file) return;
    if (!file.type.startsWith('image/')) {
      showToast('이미지 파일(강의자료 캡처, 필기 사진 등)을 업로드해주세요.');
      return;
    }
    const reader = new FileReader();
    reader.onload = () => {
      const dataUrl = reader.result;
      setAiImagePreview(dataUrl);
      const base64 = dataUrl.split(',')[1];
      setAiImageBase64(base64);
      setAiImageMime(file.type);
      showToast('강의자료/족보 이미지가 첨부되었습니다. AI 분석을 실행해보세요!');
    };
    reader.readAsDataURL(file);
  };

  const callGeminiCoach = async (customPromptText = null) => {
    const queryToUse = customPromptText || aiPrompt;
    if (!queryToUse.trim() && !aiImageBase64) {
      showToast('질문이나 강의노트 내용을 입력하거나 이미지를 첨부해주세요.');
      return;
    }

    setAiLoading(true);
    setAiResponse('');

    const subjectInfo = EXAMS.find((ex) => ex.id === aiSubject);
    const systemInstruction = `당신은 대한민국 최상위 전자전기공학부 전담 AI 시험 전략 코치입니다.
현재 학생의 시험 일정은 다음과 같습니다:
1. 2026년 10월 21일(수) 09:00~10:00: 임베디드 시스템 (ATmega128 레지스터, GPIO, 인터럽트, 타이머/카운터, UART, ADC)
2. 2026년 10월 23일(금) 09:00: 전자반도체공학기초 (교재 8, 9단원 기반 오픈북 테스트)
3. 2026년 10월 26일(월) 09:00~12:00: 회로이론 (난이도 극악 HELL, 3시간 시험)
4. 2026년 10월 26일(월) 12:00~15:00: 전자기학 (난이도 극악 HELL, 회로이론 직후 3시간 연강)

현재 선택된 과목은 [${subjectInfo?.name}] 입니다.
학생이 과거 정리했던 파일 내용, 족보, 강의록 텍스트 또는 이미지를 보내면:
- 시험에 바로 나오는 핵심 공식/레지스터/유도 과정을 가독성 높게 요약하고,
- 실전 시험에서 실수하기 쉬운 함정 포인트와 예상 문제(풀이 포함)를 명확하고 간결하게 제시하세요.`;

    const parts = [{ text: queryToUse || '첨부된 강의자료/노트 이미지를 분석해서 시험 핵심 요약과 예상 문제를 만들어줘.' }];
    if (aiImageBase64) {
      parts.push({
        inlineData: {
          mimeType: aiImageMime,
          data: aiImageBase64
        }
      });
    }

    const payload = {
      contents: [{ role: 'user', parts }],
      systemInstruction: { parts: [{ text: systemInstruction }] }
    };

    const apiKey = "";
    const resolvedKey = customApiKey.trim() || apiKey;
    const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${resolvedKey}`;

    const fetchWithRetry = async (retries = 3, delay = 1000) => {
      try {
        const res = await fetch(apiUrl, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });
        if (!res.ok) throw new Error(`HTTP_${res.status}`);
        return await res.json();
      } catch (err) {
        if (retries > 0) {
          await new Promise((r) => setTimeout(r, delay));
          return fetchWithRetry(retries - 1, delay * 2);
        }
        throw err;
      }
    };

    try {
      const result = await fetchWithRetry();
      const text = result?.candidates?.[0]?.content?.parts?.[0]?.text;
      if (text) {
        setAiResponse(text);
      } else {
        setAiResponse('응답을 생성하지 못했습니다. 다시 시도해주세요.');
      }
    } catch (err) {
      setAiResponse(
        'API 호출에 실패했습니다. 만약 GitHub 등 외부 환경에 배포하셨다면 우측 상단 [클라우드 저장 & GitHub 설정] 버튼을 눌러 무료 Gemini API 키를 입력해주세요!'
      );
    } finally {
      setAiLoading(false);
    }
  };

  const getSubjectStyle = (subjectKey) => {
    switch (subjectKey) {
      case 'embedded':
        return {
          label: '임베디드 (ATmega128)',
          badge: 'bg-sky-500/15 text-sky-300 border-sky-500/25',
          cardBorder: 'border-l-[3px] border-l-sky-400 border-white/[0.08] bg-gradient-to-r from-sky-500/[0.08] to-white/[0.02]',
          icon: <Cpu className="w-3.5 h-3.5 text-sky-400" />
        };
      case 'semicon':
        return {
          label: '전자반도체 (8·9장 오픈북)',
          badge: 'bg-emerald-500/15 text-emerald-300 border-emerald-500/25',
          cardBorder: 'border-l-[3px] border-l-emerald-400 border-white/[0.08] bg-gradient-to-r from-emerald-500/[0.08] to-white/[0.02]',
          icon: <Layers className="w-3.5 h-3.5 text-emerald-400" />
        };
      case 'circuit':
        return {
          label: '회로이론 [HELL 🔥]',
          badge: 'bg-rose-500/15 text-rose-300 border-rose-500/30',
          cardBorder: 'border-l-[3px] border-l-rose-500 border-white/[0.08] bg-gradient-to-r from-rose-500/[0.10] to-white/[0.02]',
          icon: <Zap className="w-3.5 h-3.5 text-rose-400" />
        };
      case 'em':
        return {
          label: '전자기학 [HELL 💀]',
          badge: 'bg-violet-500/15 text-violet-300 border-violet-500/30',
          cardBorder: 'border-l-[3px] border-l-violet-500 border-white/[0.08] bg-gradient-to-r from-violet-500/[0.10] to-white/[0.02]',
          icon: <Magnet className="w-3.5 h-3.5 text-violet-400" />
        };
      case 'rest':
      default:
        return {
          label: '리커버리 & 휴식 보장 ☕',
          badge: 'bg-amber-500/15 text-amber-300 border-amber-500/25',
          cardBorder: 'border-l-[3px] border-l-amber-400 border-white/[0.06] bg-gradient-to-r from-amber-500/[0.06] to-white/[0.01]',
          icon: <Coffee className="w-3.5 h-3.5 text-amber-400" />
        };
    }
  };

  const currentDaySchedule = WEEKLY_SCHEDULE[selectedDayIdx];
  const ringRadius = 54;
  const ringCircumference = 2 * Math.PI * ringRadius;
  const ringStrokeDashoffset = ringCircumference - (timerProgressPercent / 100) * ringCircumference;

  return (
    <div
      className="min-h-screen bg-[#000000] text-zinc-100 antialiased selection:bg-rose-500/30 selection:text-rose-200 pb-28"
      style={{
        fontFamily:
          '-apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "Pretendard", "Inter", sans-serif'
      }}
    >
      {/* Apple Spatial Wallpaper Ambient Mesh */}
      <div className="fixed inset-0 pointer-events-none overflow-hidden z-0">
        <div className="absolute -top-48 left-1/4 w-[650px] h-[420px] bg-rose-500/[0.08] rounded-full blur-[150px]" />
        <div className="absolute top-1/4 -right-32 w-[550px] h-[550px] bg-violet-600/[0.08] rounded-full blur-[160px]" />
        <div className="absolute -bottom-48 left-10 w-[550px] h-[450px] bg-sky-500/[0.07] rounded-full blur-[150px]" />
      </div>

      {/* Apple iOS Dynamic Island Toast Notification */}
      {toastMsg && (
        <div className="fixed top-4 left-1/2 -translate-x-1/2 z-50 flex items-center gap-3 px-5 py-3 rounded-full bg-zinc-900/95 border border-white/15 shadow-[0_20px_50px_rgba(0,0,0,0.8)] backdrop-blur-2xl animate-bounce">
          <div className="w-6 h-6 rounded-full bg-amber-400/20 flex items-center justify-center">
            <Sparkles className="w-3.5 h-3.5 text-amber-300 shrink-0" />
          </div>
          <span className="text-xs sm:text-sm font-medium tracking-tight text-white">{toastMsg}</span>
        </div>
      )}

      {/* Apple StandBy Fullscreen Mode */}
      {isStandbyMode && (
        <div className="fixed inset-0 z-50 bg-black/95 backdrop-blur-3xl flex flex-col items-center justify-between p-6 sm:p-12">
          <div className="w-full max-w-5xl flex items-center justify-between">
            <div className="flex items-center gap-3">
              <span className="w-2.5 h-2.5 rounded-full bg-rose-500 animate-pulse" />
              <span className="text-xs sm:text-sm font-semibold tracking-tight text-zinc-400">
                STANDBY FOCUS MODE · {timerTheme.name}
              </span>
            </div>
            <button
              onClick={() => setIsStandbyMode(false)}
              className="px-4 py-2 rounded-full bg-white/10 hover:bg-white/15 border border-white/10 text-xs font-semibold text-white flex items-center gap-1.5 transition-all"
            >
              <X className="w-4 h-4" />
              <span>스탠바이 종료</span>
            </button>
          </div>

          <div className="flex flex-col lg:flex-row items-center justify-center gap-10 lg:gap-16 my-auto">
            <div className="relative w-64 h-64 sm:w-80 sm:h-80 flex items-center justify-center">
              <svg className="w-full h-full -rotate-90" viewBox="0 0 128 128">
                <circle
                  cx="64"
                  cy="64"
                  r={ringRadius}
                  stroke="rgba(255,255,255,0.07)"
                  strokeWidth="8"
                  fill="none"
                />
                <circle
                  cx="64"
                  cy="64"
                  r={ringRadius}
                  stroke={timerTheme.stroke}
                  strokeWidth="8"
                  strokeDasharray={ringCircumference}
                  strokeDashoffset={ringStrokeDashoffset}
                  strokeLinecap="round"
                  fill="none"
                  className="transition-all duration-700 ease-out"
                />
              </svg>
              <div className="absolute inset-0 flex flex-col items-center justify-center text-center">
                <span className="text-xs font-semibold uppercase tracking-widest text-zinc-400">
                  진행률
                </span>
                <span className="text-4xl sm:text-5xl font-bold tabular-nums text-white mt-1">
                  {timerProgressPercent}%
                </span>
                <span className="text-[11px] text-zinc-400 mt-1">
                  종료 예정 {estimatedEndTimeString}
                </span>
              </div>
            </div>

            <div className="text-center lg:text-left space-y-6">
              <div className="inline-flex items-center gap-2 px-3.5 py-1.5 rounded-full border text-xs font-semibold bg-white/[0.04] border-white/10 text-zinc-300">
                <Clock className="w-3.5 h-3.5" style={{ color: timerTheme.stroke }} />
                <span>{timerTheme.subtitle}</span>
              </div>

              <div
                className="text-6xl sm:text-8xl lg:text-9xl font-extralight tracking-tighter tabular-nums text-white leading-none select-none"
                style={{ fontVariantNumeric: 'tabular-nums' }}
              >
                {formatTimer(timeLeft)}
              </div>

              <div className="flex flex-wrap items-center justify-center lg:justify-start gap-3">
                <button
                  onClick={() => setIsRunning(!isRunning)}
                  className={`px-8 py-4 rounded-full text-sm sm:text-base font-semibold flex items-center gap-2.5 transition-all shadow-2xl ${
                    isRunning
                      ? 'bg-amber-400 text-zinc-950'
                      : `${timerTheme.button} shadow-lg`
                  }`}
                >
                  {isRunning ? <Pause className="w-5 h-5 fill-current" /> : <Play className="w-5 h-5 fill-current" />}
                  <span>{isRunning ? '일시정지' : '집중 타이머 시작'}</span>
                </button>
                <button
                  onClick={() => adjustTimerMinutes(-5)}
                  className="px-4 py-4 rounded-full bg-white/[0.07] hover:bg-white/[0.12] border border-white/10 text-sm font-semibold text-zinc-200"
                >
                  -5분
                </button>
                <button
                  onClick={() => adjustTimerMinutes(5)}
                  className="px-4 py-4 rounded-full bg-white/[0.07] hover:bg-white/[0.12] border border-white/10 text-sm font-semibold text-zinc-200"
                >
                  +5분
                </button>
                <button
                  onClick={() => selectTimerPreset(timerMode)}
                  className="p-4 rounded-full bg-white/[0.07] hover:bg-white/[0.12] border border-white/10 text-zinc-300"
                  title="초기화"
                >
                  <RotateCcw className="w-5 h-5" />
                </button>
              </div>
            </div>
          </div>

          <div className="flex flex-wrap items-center justify-center gap-2 bg-white/[0.05] p-1.5 rounded-full border border-white/10">
            {[
              { id: 'focus50', label: '50분 집중' },
              { id: 'deep', label: '90분 딥워크' },
              { id: 'hell', label: '180분 실전(회로·전자기)' },
              { id: 'rest', label: '15분 리커버리' }
            ].map((m) => (
              <button
                key={m.id}
                onClick={() => selectTimerPreset(m.id)}
                className={`px-4 py-2 rounded-full text-xs font-semibold transition-all ${
                  timerMode === m.id
                    ? 'bg-white text-zinc-950 shadow'
                    : 'text-zinc-400 hover:text-white'
                }`}
              >
                {m.label}
              </button>
            ))}
          </div>
        </div>
      )}

      {/* Top Apple Glass Navigation Bar */}
      <header className="sticky top-0 z-30 backdrop-blur-2xl bg-black/70 border-b border-white/[0.08]">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 flex flex-col lg:flex-row lg:items-center lg:justify-between gap-3.5">
          <div className="flex items-center justify-between">
            <div className="flex items-center gap-3.5">
              <div className="w-10 h-10 rounded-2xl bg-gradient-to-b from-zinc-700 to-zinc-900 p-[1px] shadow-inner">
                <div className="w-full h-full bg-zinc-950 rounded-[15px] flex items-center justify-center">
                  <Flame className="w-5 h-5 text-rose-400" />
                </div>
              </div>
              <div>
                <div className="flex items-center gap-2.5 flex-wrap">
                  <h1 className="text-base sm:text-lg font-semibold tracking-tight text-white">
                    Exam Sprint Pro
                  </h1>
                  <span className="text-[11px] font-medium px-2.5 py-0.5 rounded-full bg-white/[0.08] text-zinc-300 border border-white/[0.08]">
                    Oct 21 – Oct 26
                  </span>
                  {isRunning && (
                    <span className="inline-flex items-center gap-1.5 text-[11px] font-mono font-semibold px-2.5 py-0.5 rounded-full bg-rose-500/20 text-rose-300 border border-rose-500/30">
                      <span className="w-1.5 h-1.5 rounded-full bg-rose-400 animate-ping" />
                      {formatTimer(timeLeft)}
                    </span>
                  )}
                </div>
                <p className="text-[11px] text-zinc-400 mt-0.5">
                  임베디드(ATmega128) · 전자반도체(8·9장) · 회로이론 · 전자기학
                </p>
              </div>
            </div>

            <div className="flex items-center gap-2 lg:hidden">
              <button
                onClick={() => setShowCloudModal(true)}
                className="flex items-center gap-1.5 px-3 py-1.5 rounded-full bg-white/[0.07] border border-white/[0.1] text-zinc-200 text-xs font-medium"
              >
                <span
                  className={`w-2 h-2 rounded-full ${
                    cloudStatus === 'synced' ? 'bg-emerald-400' : 'bg-amber-400'
                  }`}
                />
                <span>동기화</span>
              </button>
              <button
                onClick={() => setSoundEnabled(!soundEnabled)}
                className="p-2 rounded-full bg-white/[0.07] border border-white/[0.1] text-zinc-300"
                title="사운드 토글"
              >
                {soundEnabled ? <Volume2 className="w-4 h-4 text-emerald-400" /> : <VolumeX className="w-4 h-4 text-zinc-500" />}
              </button>
            </div>
          </div>

          <div className="flex items-center justify-between lg:justify-end gap-3">
            <nav className="flex items-center p-1 rounded-full bg-zinc-900/90 border border-white/[0.08] shadow-inner overflow-x-auto">
              {[
                { id: 'dashboard', label: '대시보드', icon: Compass },
                { id: 'schedule', label: '8일 캘린더', icon: Calendar },
                { id: 'vault', label: '단권화·오픈북', icon: BookOpen },
                { id: 'ai', label: 'AI 족보 코치', icon: Sparkles }
              ].map((tab) => {
                const Icon = tab.icon;
                const isActive = activeTab === tab.id;
                return (
                  <button
                    key={tab.id}
                    onClick={() => setActiveTab(tab.id)}
                    className={`flex items-center gap-1.5 px-4 py-1.5 rounded-full text-xs font-semibold tracking-tight transition-all whitespace-nowrap ${
                      isActive
                        ? 'bg-white text-zinc-950 shadow-sm'
                        : 'text-zinc-400 hover:text-white'
                    }`}
                  >
                    <Icon className="w-3.5 h-3.5" />
                    <span>{tab.label}</span>
                  </button>
                );
              })}
            </nav>

            <div className="hidden lg:flex items-center gap-2">
              <button
                onClick={() => setSoundEnabled(!soundEnabled)}
                className="p-2 rounded-full bg-zinc-900/90 hover:bg-zinc-800 border border-white/[0.08] text-zinc-300 transition-all"
                title="타이머 알림음"
              >
                {soundEnabled ? <Volume2 className="w-4 h-4 text-emerald-400" /> : <VolumeX className="w-4 h-4 text-zinc-500" />}
              </button>
              <button
                onClick={() => setShowCloudModal(true)}
                className="flex items-center gap-2 px-3.5 py-2 rounded-full bg-zinc-900/90 hover:bg-zinc-800 border border-white/[0.1] text-xs font-medium text-zinc-200 transition-all shrink-0"
              >
                <span
                  className={`w-2 h-2 rounded-full ${
                    cloudStatus === 'synced'
                      ? 'bg-emerald-400 shadow-[0_0_8px_#34d399]'
                      : cloudStatus === 'saving'
                      ? 'bg-amber-400 animate-ping'
                      : 'bg-zinc-500'
                  }`}
                />
                <Cloud className="w-3.5 h-3.5 text-zinc-300" />
                <span>
                  {activeSyncRoom
                    ? `Sync: ${activeSyncRoom}`
                    : cloudStatus === 'synced'
                    ? 'iCloud급 자동저장됨'
                    : '저장소 & API 설정'}
                </span>
              </button>
            </div>
          </div>
        </div>
      </header>

      {/* Main Content Container */}
      <main className="relative z-10 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-6 space-y-6">
        {/* APPLE STANDBY & WATCH RING STYLE FOCUS TIMER HERO WIDGET */}
        <section className="relative overflow-hidden rounded-[28px] bg-zinc-900/70 backdrop-blur-2xl border border-white/[0.09] shadow-[inset_0_1px_0_0_rgba(255,255,255,0.12),0_20px_50px_rgba(0,0,0,0.6)] p-5 sm:p-7">
          <div
            className="absolute -top-24 right-1/4 w-80 h-48 rounded-full blur-[90px] pointer-events-none opacity-30 transition-all duration-700"
            style={{ backgroundColor: timerTheme.stroke }}
          />

          <div className="relative z-10 grid grid-cols-1 lg:grid-cols-12 gap-6 items-center">
            <div className="lg:col-span-7 flex flex-col sm:flex-row items-center gap-6">
              <div
                onClick={() => setIsRunning(!isRunning)}
                className="relative w-32 h-32 sm:w-36 sm:h-36 shrink-0 flex items-center justify-center cursor-pointer group"
                title="클릭하여 시작/일시정지"
              >
                <svg className="w-full h-full -rotate-90" viewBox="0 0 128 128">
                  <circle
                    cx="64"
                    cy="64"
                    r={ringRadius}
                    stroke="rgba(255,255,255,0.08)"
                    strokeWidth="9"
                    fill="none"
                  />
                  <circle
                    cx="64"
                    cy="64"
                    r={ringRadius}
                    stroke={timerTheme.stroke}
                    strokeWidth="9"
                    strokeDasharray={ringCircumference}
                    strokeDashoffset={ringStrokeDashoffset}
                    strokeLinecap="round"
                    fill="none"
                    className="transition-all duration-700 ease-out"
                  />
                </svg>
                <div className="absolute inset-0 flex flex-col items-center justify-center text-center">
                  <span className="text-[10px] font-semibold uppercase tracking-widest text-zinc-400">
                    {isRunning ? 'RUNNING' : 'READY'}
                  </span>
                  <span className="text-2xl font-bold tabular-nums text-white mt-0.5">
                    {timerProgressPercent}%
                  </span>
                  <span className="text-[10px] text-zinc-400 group-hover:text-white transition-colors mt-0.5">
                    {isRunning ? '일시정지' : '터치해 시작'}
                  </span>
                </div>
              </div>

              <div className="flex-1 text-center sm:text-left">
                <div className="flex flex-wrap items-center justify-center sm:justify-start gap-2">
                  <span className={`text-[11px] font-semibold px-2.5 py-0.5 rounded-full border ${timerTheme.badge}`}>
                    {timerTheme.name}
                  </span>
                  <span className="text-[11px] font-medium text-zinc-400 bg-white/[0.05] px-2.5 py-0.5 rounded-full border border-white/[0.06] tabular-nums">
                    예상 종료 {estimatedEndTimeString}
                  </span>
                </div>

                <div
                  className="text-5xl sm:text-6xl lg:text-7xl font-light tracking-tight text-white tabular-nums mt-2 leading-none select-none"
                  style={{ fontVariantNumeric: 'tabular-nums' }}
                >
                  {formatTimer(timeLeft)}
                </div>

                <div className="mt-3.5 w-full max-w-md mx-auto sm:mx-0 h-1.5 rounded-full bg-white/[0.08] overflow-hidden">
                  <div
                    className="h-full rounded-full transition-all duration-500"
                    style={{
                      width: `${Math.max(2, timerProgressPercent)}%`,
                      backgroundColor: timerTheme.stroke
                    }}
                  />
                </div>

                <p className="text-xs text-zinc-400 mt-2">
                  {timerTheme.subtitle}
                </p>
              </div>
            </div>

            <div className="lg:col-span-5 flex flex-col sm:items-end justify-between gap-4 border-t lg:border-t-0 lg:border-l border-white/[0.08] pt-4 lg:pt-0 lg:pl-6">
              <div className="w-full grid grid-cols-2 sm:grid-cols-4 gap-1.5 bg-black/50 p-1.5 rounded-2xl border border-white/[0.07]">
                {[
                  { id: 'focus50', label: '50분 집중', sub: '단권화' },
                  { id: 'deep', label: '90분 딥워크', sub: '킬러문항' },
                  { id: 'hell', label: '180분 실전', sub: '회로·전자기' },
                  { id: 'rest', label: '15분 휴식', sub: '리커버리' }
                ].map((m) => {
                  const active = timerMode === m.id;
                  return (
                    <button
                      key={m.id}
                      onClick={() => selectTimerPreset(m.id)}
                      className={`py-2 px-2 rounded-xl text-center transition-all ${
                        active
                          ? 'bg-white text-zinc-950 shadow-sm font-semibold'
                          : 'text-zinc-400 hover:text-zinc-200 hover:bg-white/[0.04]'
                      }`}
                    >
                      <div className="text-xs leading-tight">{m.label}</div>
                      <div className={`text-[10px] mt-0.5 ${active ? 'text-zinc-600' : 'text-zinc-500'}`}>
                        {m.sub}
                      </div>
                    </button>
                  );
                })}
              </div>

              <div className="w-full flex flex-wrap items-center justify-between sm:justify-end gap-2">
                <div className="flex items-center gap-1 bg-white/[0.05] p-1 rounded-full border border-white/[0.08]">
                  <button
                    onClick={() => adjustTimerMinutes(-5)}
                    className="px-2.5 py-1.5 rounded-full text-xs font-medium text-zinc-300 hover:bg-white/10 transition-colors"
                    title="5분 감소"
                  >
                    -5분
                  </button>
                  <button
                    onClick={() => adjustTimerMinutes(5)}
                    className="px-2.5 py-1.5 rounded-full text-xs font-medium text-zinc-300 hover:bg-white/10 transition-colors"
                    title="5분 추가"
                  >
                    +5분
                  </button>
                </div>

                <button
                  onClick={() => setIsRunning(!isRunning)}
                  className={`flex-1 sm:flex-initial px-6 py-2.5 rounded-full text-xs sm:text-sm font-semibold flex items-center justify-center gap-2 transition-all shadow-lg ${
                    isRunning
                      ? 'bg-amber-400 text-zinc-950 shadow-amber-500/20'
                      : `${timerTheme.button} ${timerTheme.glow}`
                  }`}
                >
                  {isRunning ? (
                    <>
                      <Pause className="w-4 h-4 fill-current" />
                      <span>일시정지</span>
                    </>
                  ) : (
                    <>
                      <Play className="w-4 h-4 fill-current" />
                      <span>타이머 시작</span>
                    </>
                  )}
                </button>

                <button
                  onClick={() => selectTimerPreset(timerMode)}
                  className="p-2.5 rounded-full bg-white/[0.06] hover:bg-white/[0.12] border border-white/[0.08] text-zinc-300 transition-colors"
                  title="타이머 리셋"
                >
                  <RotateCcw className="w-4 h-4" />
                </button>

                <button
                  onClick={() => setIsStandbyMode(true)}
                  className="px-3.5 py-2.5 rounded-full bg-white/[0.06] hover:bg-white/[0.12] border border-white/[0.1] text-xs font-medium text-zinc-200 flex items-center gap-1.5 transition-all"
                  title="책상 위 전체화면 시계 모드"
                >
                  <Clock className="w-3.5 h-3.5 text-amber-300" />
                  <span>스탠바이 모드</span>
                </button>
              </div>
            </div>
          </div>
        </section>

        {activeTab === 'dashboard' && (
          <div className="space-y-6">
            <div className="rounded-[26px] bg-gradient-to-r from-rose-500/[0.12] via-violet-500/[0.08] to-zinc-900/60 backdrop-blur-2xl border border-rose-500/25 p-5 sm:p-6 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 shadow-[inset_0_1px_0_0_rgba(255,255,255,0.1)]">
              <div className="flex items-start gap-4">
                <div className="w-11 h-11 rounded-2xl bg-rose-500/20 border border-rose-500/30 text-rose-300 flex items-center justify-center shrink-0 mt-0.5">
                  <ShieldAlert className="w-5 h-5" />
                </div>
                <div>
                  <div className="flex flex-wrap items-center gap-2">
                    <span className="text-[11px] font-semibold uppercase tracking-wider px-2.5 py-0.5 rounded-full bg-rose-500 text-white">
                      INTERLEAVING STRATEGY
                    </span>
                    <h2 className="text-base sm:text-lg font-semibold tracking-tight text-white">
                      10/26(월) 6시간 연강(회로이론+전자기학) 대비 분산 학습 원칙
                    </h2>
                  </div>
                  <p className="text-xs sm:text-sm text-zinc-300 mt-1.5 leading-relaxed">
                    월요일 <strong className="text-rose-300">09:00~12:00 회로이론</strong> 직후 쉬는 시간 없이{' '}
                    <strong className="text-violet-300">12:00~15:00 전자기학</strong>이 이어집니다. 금요일 오후부터 시작하면 절대 시간이 부족하기 때문에{' '}
                    <span className="text-white font-medium underline decoration-rose-400/70">
                      월~목요일에도 매일 40~50%의 시간을 회로·전자기학 킬러 문제 풀이에 선투자
                    </span>
                    하도록 설계했습니다.
                  </p>
                </div>
              </div>
              <button
                onClick={() => {
                  setScheduleViewMode('calendar');
                  setActiveTab('schedule');
                }}
                className="shrink-0 flex items-center gap-2 px-4 py-2.5 rounded-full bg-white text-zinc-950 hover:bg-zinc-200 text-xs font-semibold transition-all shadow-md"
              >
                <Calendar className="w-3.5 h-3.5" />
                <span>주간 타임캘린더 열기</span>
                <ChevronRight className="w-3.5 h-3.5" />
              </button>
            </div>

            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {EXAMS.map((exam) => {
                const subjProg = progressMetrics.subjectProgress[exam.id] || 0;
                return (
                  <div
                    key={exam.id}
                    className="group rounded-[28px] bg-zinc-900/65 backdrop-blur-2xl border border-white/[0.08] hover:border-white/[0.16] p-6 flex flex-col justify-between transition-all duration-300 shadow-[inset_0_1px_0_0_rgba(255,255,255,0.1)]"
                  >
                    <div>
                      <div className="flex items-center justify-between gap-2 flex-wrap">
                        <span className={`text-[11px] font-semibold px-3 py-1 rounded-full border ${exam.badgeBg}`}>
                          {exam.difficultyLabel}
                        </span>
                        <span className="text-xs font-mono font-medium text-zinc-300 bg-white/[0.05] px-3 py-1 rounded-full border border-white/[0.08] tabular-nums">
                          {getDiffString(exam.date)}
                        </span>
                      </div>

                      <h3 className="text-lg sm:text-xl font-semibold tracking-tight text-white mt-4">
                        {exam.name}
                      </h3>
                      <p className="text-xs text-zinc-400 mt-1.5 flex items-center gap-2 flex-wrap">
                        <Calendar className="w-3.5 h-3.5 text-zinc-500" />
                        <span>{exam.dayLabel}</span>
                        <span className="text-zinc-700">•</span>
                        <span className={exam.textAccent}>{exam.type}</span>
                      </p>

                      <div className="mt-4 p-4 rounded-2xl bg-black/40 border border-white/[0.05] text-xs text-zinc-300 leading-relaxed">
                        <span className="text-white font-semibold block mb-1">핵심 공략 포인트</span>
                        {exam.strategy}
                      </div>
                    </div>

                    <div className="mt-5 pt-4 border-t border-white/[0.07]">
                      <div className="flex items-center justify-between text-xs mb-2">
                        <span className="text-zinc-400 font-medium">단원 마스터리</span>
                        <span className="font-mono font-semibold text-white tabular-nums">{subjProg}%</span>
                      </div>
                      <div className="w-full h-1.5 rounded-full bg-white/[0.07] overflow-hidden">
                        <div
                          className="h-full rounded-full transition-all duration-500"
                          style={{ width: `${subjProg}%`, backgroundColor: exam.accentHex }}
                        />
                      </div>

                      <div className="mt-4 flex items-center justify-between gap-2">
                        <button
                          onClick={() => {
                            setActiveVaultSubject(exam.id);
                            setActiveTab('vault');
                          }}
                          className="text-xs font-medium text-zinc-200 hover:text-white flex items-center gap-1.5 bg-white/[0.06] hover:bg-white/[0.12] px-3.5 py-2 rounded-full border border-white/[0.08] transition-all"
                        >
                          <CheckCircle2 className="w-3.5 h-3.5" />
                          <span>단권화 체크리스트</span>
                        </button>

                        {(exam.id === 'embedded' || exam.id === 'semicon') && (
                          <button
                            onClick={() => setCheatSheetModal(exam.id)}
                            className="text-xs font-medium text-amber-300 hover:text-amber-200 flex items-center gap-1.5 bg-amber-500/10 hover:bg-amber-500/20 px-3.5 py-2 rounded-full border border-amber-500/25 transition-all"
                          >
                            <Bookmark className="w-3.5 h-3.5" />
                            <span>{exam.id === 'embedded' ? '레지스터 요약' : '오픈북 전략'}</span>
                          </button>
                        )}
                      </div>
                    </div>
                  </div>
                );
              })}
            </div>

            <div className="rounded-[28px] bg-zinc-900/65 backdrop-blur-2xl border border-white/[0.08] p-6 shadow-[inset_0_1px_0_0_rgba(255,255,255,0.1)]">
              <div className="flex flex-col sm:flex-row sm:items-center justify-between gap-2 mb-5">
                <div>
                  <h3 className="text-base sm:text-lg font-semibold tracking-tight text-white flex items-center gap-2">
                    <BarChart3 className="w-5 h-5 text-sky-400" />
                    <span>8일간 과목별 에너지 밸런스 (10/19 월 ~ 10/26 월)</span>
                  </h3>
                  <p className="text-xs text-zinc-400 mt-0.5">
                    날짜 카드를 선택하면 해당 요일의 시간대별 상세 타임라인으로 이동합니다.
                  </p>
                </div>
                <div className="flex flex-wrap items-center gap-3 text-xs">
                  <span className="flex items-center gap-1.5 text-sky-300">
                    <span className="w-2 h-2 rounded-full bg-sky-400" /> 임베디드
                  </span>
                  <span className="flex items-center gap-1.5 text-emerald-300">
                    <span className="w-2 h-2 rounded-full bg-emerald-400" /> 전자반도체
                  </span>
                  <span className="flex items-center gap-1.5 text-rose-300">
                    <span className="w-2 h-2 rounded-full bg-rose-500" /> 회로이론
                  </span>
                  <span className="flex items-center gap-1.5 text-violet-300">
                    <span className="w-2 h-2 rounded-full bg-violet-500" /> 전자기학
                  </span>
                </div>
              </div>

              <div className="grid grid-cols-2 sm:grid-cols-4 lg:grid-cols-8 gap-2.5">
                {WEEKLY_SCHEDULE.map((day, idx) => {
                  const r = day.focusRatio;
                  const dayDoneCount = day.blocks.filter((b) => completedBlocks[b.id]).length;
                  return (
                    <button
                      key={day.date}
                      onClick={() => {
                        setSelectedDayIdx(idx);
                        setScheduleViewMode('daily');
                        setActiveTab('schedule');
                      }}
                      className="group text-left p-3.5 rounded-2xl bg-black/40 hover:bg-white/[0.06] border border-white/[0.07] hover:border-white/[0.18] transition-all flex flex-col justify-between"
                    >
                      <div>
                        <div className="flex items-center justify-between">
                          <span className="text-xs font-semibold text-white group-hover:text-rose-300 transition-colors">
                            {day.date.slice(5).replace('-', '.')} ({day.dayShort})
                          </span>
                          <span className="text-[10px] font-mono text-zinc-400 tabular-nums">
                            {dayDoneCount}/{day.blocks.length}
                          </span>
                        </div>
                        <p className="text-[11px] text-zinc-400 mt-1.5 line-clamp-2 leading-snug">
                          {day.dayTitle.split('·')[1] || day.dayTitle}
                        </p>
                      </div>

                      <div className="mt-3 w-full h-1.5 rounded-full bg-white/[0.07] overflow-hidden flex">
                        {r.embedded > 0 && (
                          <div style={{ width: `${r.embedded}%` }} className="bg-sky-400 h-full" />
                        )}
                        {r.semicon > 0 && (
                          <div style={{ width: `${r.semicon}%` }} className="bg-emerald-400 h-full" />
                        )}
                        {r.circuit > 0 && (
                          <div style={{ width: `${r.circuit}%` }} className="bg-rose-500 h-full" />
                        )}
                        {r.em > 0 && (
                          <div style={{ width: `${r.em}%` }} className="bg-violet-500 h-full" />
                        )}
                      </div>
                    </button>
                  );
                })}
              </div>
            </div>
          </div>
        )}

        {activeTab === 'schedule' && (
          <div className="space-y-6">
            <div className="rounded-3xl bg-[#11131F] border border-white/10 p-4 sm:p-5 flex flex-col lg:flex-row lg:items-center justify-between gap-4">
              <div>
                <div className="flex items-center gap-2">
                  <span className="text-xs font-bold px-2.5 py-0.5 rounded-full bg-rose-500/20 text-rose-300 border border-rose-500/30">
                    8-DAY MASTER PLAN
                  </span>
                  <h2 className="text-base sm:text-lg font-bold text-white">
                    10/19(월) ~ 10/26(월) 전략 스케줄 캘린더
                  </h2>
                </div>
                <p className="text-xs text-slate-400 mt-1">
                  일주일 전체 흐름을 달력으로 한눈에 비교하거나, 요일별 상세 타임라인으로 전환할 수 있습니다. 블록을 클릭하면 완료 체크가 됩니다.
                </p>
              </div>

              <div className="flex flex-wrap items-center gap-2">
                <div className="flex items-center bg-[#0B0D16] p-1 rounded-2xl border border-white/10">
                  <button
                    onClick={() => setScheduleViewMode('calendar')}
                    className={`px-3.5 py-2 rounded-xl text-xs font-bold transition-all flex items-center gap-1.5 ${
                      scheduleViewMode === 'calendar'
                        ? 'bg-gradient-to-r from-rose-500 to-violet-600 text-white shadow'
                        : 'text-slate-400 hover:text-white'
                    }`}
                  >
                    <Calendar className="w-3.5 h-3.5" />
                    <span>일주일 한눈에 보기 (주간 달력)</span>
                  </button>
                  <button
                    onClick={() => setScheduleViewMode('daily')}
                    className={`px-3.5 py-2 rounded-xl text-xs font-bold transition-all flex items-center gap-1.5 ${
                      scheduleViewMode === 'daily'
                        ? 'bg-gradient-to-r from-rose-500 to-violet-600 text-white shadow'
                        : 'text-slate-400 hover:text-white'
                    }`}
                  >
                    <CheckCircle2 className="w-3.5 h-3.5" />
                    <span>요일별 집중 타임라인</span>
                  </button>
                </div>
              </div>
            </div>

            {scheduleViewMode === 'calendar' ? (
              <div className="space-y-4">
                <div className="rounded-2xl bg-[#11131F] border border-white/10 p-3.5 flex flex-col md:flex-row md:items-center justify-between gap-3">
                  <div className="flex flex-wrap items-center gap-1.5">
                    <span className="text-xs font-semibold text-slate-400 mr-1">과목 하이라이트 필터:</span>
                    {[
                      { id: 'all', label: '전체 보기', color: 'bg-white/15 text-white border-white/20' },
                      { id: 'embedded', label: `임베디드 (${progressMetrics.subjectHours.embedded}h)`, color: 'bg-cyan-500/20 text-cyan-300 border-cyan-500/40' },
                      { id: 'semicon', label: `전자반도체 (${progressMetrics.subjectHours.semicon}h)`, color: 'bg-emerald-500/20 text-emerald-300 border-emerald-500/40' },
                      { id: 'circuit', label: `회로이론 HELL (${progressMetrics.subjectHours.circuit}h)`, color: 'bg-rose-500/20 text-rose-300 border-rose-500/40' },
                      { id: 'em', label: `전자기학 HELL (${progressMetrics.subjectHours.em}h)`, color: 'bg-violet-500/20 text-violet-300 border-violet-500/40' },
                      { id: 'rest', label: `보장된 휴식 (${progressMetrics.subjectHours.rest}h)`, color: 'bg-amber-500/20 text-amber-300 border-amber-500/40' }
                    ].map((f) => (
                      <button
                        key={f.id}
                        onClick={() => setCalendarFilter(f.id)}
                        className={`px-3 py-1.5 rounded-xl text-xs font-bold border transition-all ${
                          calendarFilter === f.id
                            ? `${f.color} ring-2 ring-white/20 scale-[1.02]`
                            : 'bg-[#0B0D16] border-white/5 text-slate-400 hover:text-slate-200'
                        }`}
                      >
                        {f.label}
                      </button>
                    ))}
                  </div>
                  <div className="text-[11px] text-slate-400">
                    💡 칸을 클릭하면 상세 전략 확인 및 완료 체크가 가능합니다.
                  </div>
                </div>

                <div className="overflow-x-auto pb-3">
                  <div className="min-w-[1120px] grid grid-cols-8 gap-2.5">
                    {WEEKLY_SCHEDULE.map((day, dIdx) => {
                      const isExamDay = dIdx === 2 || dIdx === 4 || dIdx === 7;
                      const doneCount = day.blocks.filter((b) => completedBlocks[b.id]).length;
                      return (
                        <div
                          key={day.date}
                          className={`rounded-2xl border flex flex-col overflow-hidden ${
                            isExamDay
                              ? 'bg-gradient-to-b from-rose-950/25 via-[#11131F] to-[#11131F] border-rose-500/40'
                              : 'bg-[#11131F] border-white/10'
                          }`}
                        >
                          <div
                            onClick={() => {
                              setSelectedDayIdx(dIdx);
                              setScheduleViewMode('daily');
                            }}
                            className="p-3 border-b border-white/10 bg-[#0B0D16]/80 cursor-pointer hover:bg-white/5 transition-colors"
                          >
                            <div className="flex items-center justify-between">
                              <span className="text-xs font-mono font-bold text-slate-300">
                                {day.date.slice(5).replace('-', '/')}
                              </span>
                              <span
                                className={`text-[10px] font-bold px-2 py-0.5 rounded-full ${
                                  isExamDay
                                    ? 'bg-rose-500 text-white'
                                    : 'bg-white/10 text-slate-300'
                                }`}
                              >
                                {day.dayShort}
                              </span>
                            </div>

                            <div className="mt-1.5 min-h-[22px] flex items-center">
                              {dIdx === 2 && (
                                <span className="text-[10px] font-bold text-cyan-300 bg-cyan-500/15 border border-cyan-500/30 px-2 py-0.5 rounded-md w-full text-center">
                                  🎯 09:00 임베디드 시험
                                </span>
                              )}
                              {dIdx === 4 && (
                                <span className="text-[10px] font-bold text-emerald-300 bg-emerald-500/15 border border-emerald-500/30 px-2 py-0.5 rounded-md w-full text-center">
                                  🎯 09:00 반도체 오픈북
                                </span>
                              )}
                              {dIdx === 7 && (
                                <span className="text-[10px] font-bold text-rose-300 bg-rose-500/20 border border-rose-500/40 px-2 py-0.5 rounded-md w-full text-center">
                                  ⚔️ 09~15시 회로+전자기
                                </span>
                              )}
                              {!isExamDay && (
                                <span className="text-[10px] text-slate-500 font-medium truncate">
                                  완료 {doneCount}/{day.blocks.length} 블록
                                </span>
                              )}
                            </div>

                            <div className="mt-2 w-full h-1.5 rounded-full bg-black overflow-hidden flex">
                              {day.focusRatio.embedded > 0 && (
                                <div style={{ width: `${day.focusRatio.embedded}%` }} className="bg-cyan-400" />
                              )}
                              {day.focusRatio.semicon > 0 && (
                                <div style={{ width: `${day.focusRatio.semicon}%` }} className="bg-emerald-400" />
                              )}
                              {day.focusRatio.circuit > 0 && (
                                <div style={{ width: `${day.focusRatio.circuit}%` }} className="bg-rose-500" />
                              )}
                              {day.focusRatio.em > 0 && (
                                <div style={{ width: `${day.focusRatio.em}%` }} className="bg-violet-500" />
                              )}
                            </div>
                          </div>

                          <div className="p-2 space-y-2 flex-1 flex flex-col">
                            {day.blocks.map((block) => {
                              const style = getSubjectStyle(block.subject);
                              const isDone = !!completedBlocks[block.id];
                              const isDimmed = calendarFilter !== 'all' && calendarFilter !== block.subject;
                              const isRealExamBlock = block.title.includes('[실전 시험') || block.title.includes('[최종 보스');
                              const hrs = getBlockDurationHours(block.time);

                              return (
                                <div
                                  key={block.id}
                                  onClick={() => setSelectedCalBlock({ ...block, dayTitle: day.dayTitle, date: day.date })}
                                  style={{ minHeight: `${Math.max(66, Math.round(hrs * 36))}px` }}
                                  className={`group relative rounded-xl p-2.5 border transition-all cursor-pointer flex flex-col justify-between ${
                                    style.cardBorder
                                  } ${
                                    isRealExamBlock ? 'ring-2 ring-amber-400/80 shadow-lg shadow-rose-500/20' : ''
                                  } ${
                                    isDone ? 'opacity-45 bg-black/40' : 'hover:scale-[1.02] hover:z-10 hover:shadow-xl'
                                  } ${
                                    isDimmed ? 'opacity-20 grayscale' : ''
                                  }`}
                                >
                                  <div>
                                    <div className="flex items-center justify-between gap-1">
                                      <span className="text-[10px] font-mono font-bold text-slate-300 bg-black/50 px-1.5 py-0.5 rounded">
                                        {block.time}
                                      </span>
                                      <button
                                        type="button"
                                        onClick={(e) => {
                                          e.stopPropagation();
                                          toggleBlock(block.id);
                                        }}
                                        className="text-slate-400 hover:text-white"
                                        title="완료 토글"
                                      >
                                        {isDone ? (
                                          <CheckCircle2 className="w-3.5 h-3.5 text-emerald-400" />
                                        ) : (
                                          <Circle className="w-3.5 h-3.5 text-slate-500 group-hover:text-slate-300" />
                                        )}
                                      </button>
                                    </div>

                                    <div
                                      className={`text-[11px] font-bold mt-1.5 leading-snug line-clamp-3 ${
                                        isDone ? 'line-through text-slate-400' : 'text-white'
                                      }`}
                                    >
                                      {block.title}
                                    </div>
                                  </div>

                                  <div className="mt-2 pt-1.5 border-t border-white/5 flex items-center justify-between text-[10px] text-slate-400">
                                    <span className="flex items-center gap-1 truncate">
                                      {style.icon}
                                      <span className="truncate">{block.subject === 'rest' ? '휴식/식사' : block.subject.toUpperCase()}</span>
                                    </span>
                                    <span className="font-mono">{hrs}h</span>
                                  </div>
                                </div>
                              );
                            })}
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>
              </div>
            ) : (
              <>
                <div className="flex items-center gap-2 overflow-x-auto pb-2">
                  {WEEKLY_SCHEDULE.map((day, idx) => {
                    const isSelected = selectedDayIdx === idx;
                    const doneCount = day.blocks.filter((b) => completedBlocks[b.id]).length;
                    const isExamDay = idx === 2 || idx === 4 || idx === 7;
                    return (
                      <button
                        key={day.date}
                        onClick={() => setSelectedDayIdx(idx)}
                        className={`relative shrink-0 px-4 py-3 rounded-2xl border text-left transition-all ${
                          isSelected
                            ? 'bg-gradient-to-br from-rose-500/25 via-violet-500/20 to-[#181B2C] border-rose-500/50 text-white shadow-lg'
                            : 'bg-[#11131F] border-white/10 text-slate-400 hover:text-slate-200 hover:bg-white/5'
                        }`}
                      >
                        <div className="flex items-center gap-2">
                          <span className="text-xs font-mono font-bold">{day.date.slice(5)}</span>
                          <span className="text-sm font-bold">({day.dayShort})</span>
                          {isExamDay && (
                            <span className="w-2 h-2 rounded-full bg-rose-500 animate-ping" />
                          )}
                        </div>
                        <div className="text-[11px] text-slate-400 mt-1">
                          완료 {doneCount}/{day.blocks.length} 블록
                        </div>
                      </button>
                    );
                  })}
                </div>

                <div className="rounded-3xl bg-[#11131F] border border-white/10 p-5 sm:p-6">
                  <div className="flex flex-col md:flex-row md:items-center justify-between gap-4 pb-5 border-b border-white/10">
                    <div>
                      <div className="flex items-center gap-2">
                        <span className="px-2.5 py-1 rounded-lg bg-white/10 text-xs font-mono font-bold text-white">
                          {currentDaySchedule.date}
                        </span>
                        <span className="text-xs font-semibold text-amber-300 flex items-center gap-1">
                          <Coffee className="w-3.5 h-3.5" /> 식사 및 뇌 휴식 블록 보장됨
                        </span>
                      </div>
                      <h2 className="text-xl sm:text-2xl font-bold text-white mt-2">
                        {currentDaySchedule.dayTitle}
                      </h2>
                      <p className="text-xs sm:text-sm text-slate-300 mt-1">
                        {currentDaySchedule.subtitle}
                      </p>
                    </div>

                    <div className="bg-[#0B0D16] p-3.5 rounded-2xl border border-white/10 shrink-0 min-w-[230px]">
                      <div className="text-[11px] text-slate-400 font-medium mb-2">오늘의 과목 집중도 배분</div>
                      <div className="w-full h-3 rounded-full bg-black overflow-hidden flex mb-2">
                        {currentDaySchedule.focusRatio.embedded > 0 && (
                          <div style={{ width: `${currentDaySchedule.focusRatio.embedded}%` }} className="bg-cyan-400" />
                        )}
                        {currentDaySchedule.focusRatio.semicon > 0 && (
                          <div style={{ width: `${currentDaySchedule.focusRatio.semicon}%` }} className="bg-emerald-400" />
                        )}
                        {currentDaySchedule.focusRatio.circuit > 0 && (
                          <div style={{ width: `${currentDaySchedule.focusRatio.circuit}%` }} className="bg-rose-500" />
                        )}
                        {currentDaySchedule.focusRatio.em > 0 && (
                          <div style={{ width: `${currentDaySchedule.focusRatio.em}%` }} className="bg-violet-500" />
                        )}
                      </div>
                      <div className="grid grid-cols-2 gap-1 text-[11px] text-slate-300 font-mono">
                        <span>임베디드: {currentDaySchedule.focusRatio.embedded}%</span>
                        <span>반도체: {currentDaySchedule.focusRatio.semicon}%</span>
                        <span className="text-rose-300">회로이론: {currentDaySchedule.focusRatio.circuit}%</span>
                        <span className="text-violet-300">전자기학: {currentDaySchedule.focusRatio.em}%</span>
                      </div>
                    </div>
                  </div>

                  <div className="mt-5 space-y-3">
                    {currentDaySchedule.blocks.map((block) => {
                      const style = getSubjectStyle(block.subject);
                      const isDone = !!completedBlocks[block.id];
                      return (
                        <div
                          key={block.id}
                          onClick={() => toggleBlock(block.id)}
                          className={`cursor-pointer rounded-2xl border p-4 transition-all flex items-start gap-3.5 ${
                            style.cardBorder
                          } ${isDone ? 'opacity-55 bg-black/20' : 'hover:bg-white/[0.04]'}`}
                        >
                          <button
                            type="button"
                            className="mt-0.5 text-slate-300 hover:text-white shrink-0"
                          >
                            {isDone ? (
                              <CheckCircle2 className="w-5 h-5 text-emerald-400" />
                            ) : (
                              <Circle className="w-5 h-5 text-slate-500" />
                            )}
                          </button>

                          <div className="flex-1 min-w-0">
                            <div className="flex flex-wrap items-center gap-2">
                              <span className="text-xs font-mono font-bold px-2.5 py-0.5 rounded-md bg-black/50 text-white border border-white/10">
                                {block.time}
                              </span>
                              <span className={`text-[11px] font-semibold px-2.5 py-0.5 rounded-full border flex items-center gap-1 ${style.badge}`}>
                                {style.icon}
                                <span>{style.label}</span>
                              </span>
                            </div>

                            <h4 className={`text-sm sm:text-base font-bold mt-2 ${isDone ? 'line-through text-slate-400' : 'text-white'}`}>
                              {block.title}
                            </h4>
                            <p className="text-xs text-slate-300 mt-1 leading-relaxed">
                              {block.desc}
                            </p>
                          </div>
                        </div>
                      );
                    })}
                  </div>
                </div>
              </>
            )}
          </div>
        )}

        {activeTab === 'vault' && (
          <div className="space-y-6">
            <div className="grid grid-cols-2 lg:grid-cols-4 gap-3">
              {EXAMS.map((ex) => {
                const active = activeVaultSubject === ex.id;
                const prog = progressMetrics.subjectProgress[ex.id] || 0;
                return (
                  <button
                    key={ex.id}
                    onClick={() => setActiveVaultSubject(ex.id)}
                    className={`p-4 rounded-2xl border text-left transition-all ${
                      active
                        ? `bg-gradient-to-br ${ex.bgGlow} ${ex.borderGlow} text-white shadow-lg`
                        : 'bg-[#11131F] border-white/10 text-slate-400 hover:text-white'
                    }`}
                  >
                    <div className="flex items-center justify-between">
                      <span className="text-xs font-bold">{ex.shortName}</span>
                      <span className="text-xs font-mono font-bold text-white">{prog}%</span>
                    </div>
                    <div className="w-full h-1.5 rounded-full bg-black/50 mt-2.5 overflow-hidden">
                      <div
                        className="h-full rounded-full"
                        style={{ width: `${prog}%`, backgroundColor: ex.accentHex }}
                      />
                    </div>
                  </button>
                );
              })}
            </div>

            <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
              <div className="lg:col-span-2 rounded-3xl bg-[#11131F] border border-white/10 p-5 sm:p-6">
                <div className="flex items-center justify-between gap-2 mb-4">
                  <div>
                    <h3 className="text-lg font-bold text-white">
                      {EXAMS.find((e) => e.id === activeVaultSubject)?.name} 핵심 정복 체크리스트
                    </h3>
                    <p className="text-xs text-slate-400 mt-0.5">
                      기출 족보나 수업 파일에서 강조된 단원 주제를 하나씩 체크하며 빈틈을 없애세요.
                    </p>
                  </div>
                </div>

                <form onSubmit={addCustomTopic} className="flex flex-col sm:flex-row gap-2 mb-5">
                  <input
                    type="text"
                    value={newTopicText}
                    onChange={(e) => setNewTopicText(e.target.value)}
                    placeholder="새로운 시험 범위 항목이나 교수님 강조 포인트 추가..."
                    className="flex-1 px-4 py-2.5 rounded-xl bg-[#0B0D16] border border-white/15 text-xs sm:text-sm text-white placeholder:text-slate-500 focus:outline-none focus:border-rose-500"
                  />
                  <div className="flex gap-2">
                    <select
                      value={newTopicTag}
                      onChange={(e) => setNewTopicTag(e.target.value)}
                      className="px-3 py-2.5 rounded-xl bg-[#0B0D16] border border-white/15 text-xs text-slate-200 focus:outline-none"
                    >
                      <option value="필수암기">필수암기</option>
                      <option value="킬러문항">킬러문항</option>
                      <option value="오픈북인덱스">오픈북인덱스</option>
                      <option value="족보빈출">족보빈출</option>
                    </select>
                    <button
                      type="submit"
                      className="px-4 py-2.5 rounded-xl bg-rose-500 hover:bg-rose-600 text-white text-xs font-bold flex items-center gap-1.5 shrink-0"
                    >
                      <Plus className="w-4 h-4" />
                      <span>추가</span>
                    </button>
                  </div>
                </form>

                <div className="space-y-2.5">
                  {(topics[activeVaultSubject] || []).map((item) => (
                    <div
                      key={item.id}
                      className={`p-3.5 rounded-2xl border transition-all flex items-center justify-between gap-3 ${
                        item.done
                          ? 'bg-black/25 border-white/5 opacity-60'
                          : 'bg-[#171A29] border-white/10 hover:border-white/20'
                      }`}
                    >
                      <div
                        onClick={() => toggleTopic(activeVaultSubject, item.id)}
                        className="flex items-center gap-3 cursor-pointer flex-1"
                      >
                        {item.done ? (
                          <CheckCircle2 className="w-5 h-5 text-emerald-400 shrink-0" />
                        ) : (
                          <Circle className="w-5 h-5 text-slate-500 shrink-0" />
                        )}
                        <span className={`text-xs sm:text-sm font-medium ${item.done ? 'line-through text-slate-400' : 'text-slate-100'}`}>
                          {item.title}
                        </span>
                      </div>

                      <div className="flex items-center gap-2 shrink-0">
                        <span className="text-[11px] font-semibold px-2.5 py-0.5 rounded-full bg-white/10 text-slate-300">
                          {item.tag}
                        </span>
                        <button
                          onClick={() => deleteTopic(activeVaultSubject, item.id)}
                          className="p-1.5 rounded-lg text-slate-500 hover:text-rose-400 hover:bg-white/5"
                          title="삭제"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    </div>
                  ))}
                </div>

                <div className="mt-8 pt-6 border-t border-white/10">
                  <div className="flex items-center justify-between mb-3">
                    <div className="flex items-center gap-2">
                      <Brain className="w-5 h-5 text-rose-400" />
                      <h4 className="text-base font-bold text-white">
                        백지 복습용 3초 암기 플래시카드 (카드를 클릭해 정답 확인)
                      </h4>
                    </div>
                    <span className="text-xs text-slate-400">시험장 입실 직전 최종 테스트용</span>
                  </div>
                  <div className="grid grid-cols-1 gap-3">
                    {(FLASHCARDS[activeVaultSubject] || []).map((card, cIdx) => {
                      const cardKey = `${activeVaultSubject}-${cIdx}`;
                      const isFlipped = !!flippedCards[cardKey];
                      return (
                        <div
                          key={cardKey}
                          onClick={() =>
                            setFlippedCards((prev) => ({ ...prev, [cardKey]: !prev[cardKey] }))
                          }
                          className={`cursor-pointer p-4 rounded-2xl border transition-all ${
                            isFlipped
                              ? 'bg-gradient-to-r from-emerald-950/40 to-[#0B0D16] border-emerald-500/40'
                              : 'bg-[#0B0D16] border-white/10 hover:border-rose-500/40'
                          }`}
                        >
                          <div className="flex items-center justify-between text-[11px] font-bold mb-1.5">
                            <span className={isFlipped ? 'text-emerald-400' : 'text-rose-400'}>
                              {isFlipped ? '✅ 정답 및 핵심 공식 (다시 클릭하면 접기)' : `Q${cIdx + 1}. 질문 (클릭하여 정답 보기)`}
                            </span>
                            <span className="text-slate-500">FLIP</span>
                          </div>
                          <p className="text-xs sm:text-sm font-semibold text-white whitespace-pre-wrap leading-relaxed">
                            {isFlipped ? card.a : card.q}
                          </p>
                        </div>
                      );
                    })}
                  </div>
                </div>
              </div>

              <div className="space-y-6">
                <div className="rounded-3xl bg-[#11131F] border border-emerald-500/30 p-5">
                  <div className="flex items-center gap-2 text-emerald-400 font-bold text-sm mb-1">
                    <BookOpen className="w-4 h-4" />
                    <span>전자반도체(8·9장) 오픈북 3초 색인기</span>
                  </div>
                  <p className="text-xs text-slate-400 mb-4">
                    10/23(금) 오픈북 시험에서 책 페이지를 넘기느라 시간을 허비하지 않도록 공식·예제 위치를 미리 기록하세요.
                  </p>

                  <form onSubmit={addOpenBookItem} className="space-y-2 mb-4">
                    <div className="grid grid-cols-3 gap-2">
                      <select
                        value={newObChapter}
                        onChange={(e) => setNewObChapter(e.target.value)}
                        className="px-2.5 py-2 rounded-xl bg-[#0B0D16] border border-white/10 text-xs text-white"
                      >
                        <option value="8단원">8단원</option>
                        <option value="9단원">9단원</option>
                      </select>
                      <input
                        type="text"
                        value={newObPage}
                        onChange={(e) => setNewObPage(e.target.value)}
                        placeholder="페이지 (예: p.214)"
                        className="col-span-2 px-3 py-2 rounded-xl bg-[#0B0D16] border border-white/10 text-xs text-white"
                      />
                    </div>
                    <input
                      type="text"
                      value={newObKeyword}
                      onChange={(e) => setNewObKeyword(e.target.value)}
                      placeholder="공식/예제 키워드 (예: 공핍층 폭 W 공식)"
                      className="w-full px-3 py-2 rounded-xl bg-[#0B0D16] border border-white/10 text-xs text-white"
                    />
                    <div className="flex gap-2">
                      <input
                        type="text"
                        value={newObNote}
                        onChange={(e) => setNewObNote(e.target.value)}
                        placeholder="주의사항 메모 (선택)"
                        className="flex-1 px-3 py-2 rounded-xl bg-[#0B0D16] border border-white/10 text-xs text-white"
                      />
                      <button
                        type="submit"
                        className="px-3.5 py-2 rounded-xl bg-emerald-500 hover:bg-emerald-600 text-slate-950 font-bold text-xs shrink-0"
                      >
                        등록
                      </button>
                    </div>
                  </form>

                  <div className="space-y-2 max-h-64 overflow-y-auto pr-1">
                    {openBookIndex.map((ob) => (
                      <div
                        key={ob.id}
                        className="p-3 rounded-xl bg-[#0B0D16] border border-white/10 flex items-start justify-between gap-2"
                      >
                        <div>
                          <div className="flex items-center gap-1.5">
                            <span className="text-[10px] font-bold px-2 py-0.5 rounded bg-emerald-500/20 text-emerald-300">
                              {ob.chapter}
                            </span>
                            <span className="text-xs font-mono font-bold text-amber-300">
                              {ob.page}
                            </span>
                          </div>
                          <div className="text-xs font-semibold text-white mt-1">{ob.keyword}</div>
                          {ob.note && <div className="text-[11px] text-slate-400 mt-0.5">{ob.note}</div>}
                        </div>
                        <button
                          onClick={() => removeOpenBookItem(ob.id)}
                          className="text-slate-500 hover:text-rose-400 p-1"
                        >
                          <X className="w-3.5 h-3.5" />
                        </button>
                      </div>
                    ))}
                  </div>
                </div>

                <div className="rounded-3xl bg-[#11131F] border border-cyan-500/30 p-5">
                  <div className="flex items-center gap-2 text-cyan-400 font-bold text-sm mb-2">
                    <Cpu className="w-4 h-4" />
                    <span>ATmega128 핵심 레지스터 요약</span>
                  </div>
                  <div className="space-y-2">
                    {CHEAT_SHEETS.embedded.map((c, i) => (
                      <div key={i} className="p-2.5 rounded-xl bg-[#0B0D16] border border-white/5">
                        <div className="text-xs font-mono font-bold text-cyan-300">{c.reg}</div>
                        <div className="text-[11px] text-slate-300 mt-0.5 leading-relaxed">{c.desc}</div>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            </div>
          </div>
        )}

        {activeTab === 'ai' && (
          <div className="grid grid-cols-1 lg:grid-cols-12 gap-6">
            <div className="lg:col-span-5 rounded-3xl bg-[#11131F] border border-white/10 p-5 sm:p-6 space-y-5">
              <div>
                <div className="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-violet-500/20 border border-violet-500/30 text-violet-300 text-xs font-semibold mb-2">
                  <Sparkles className="w-3.5 h-3.5" />
                  <span>Gemini 3 Flash 기반 시험 족보·강의록 분석기</span>
                </div>
                <h3 className="text-lg font-bold text-white">
                  수업 자료·필기·족보 붙여넣기 & 예상문제 추출
                </h3>
                <p className="text-xs text-slate-400 mt-1 leading-relaxed">
                  가지고 계신 강의자료 텍스트를 붙여넣거나 필기/교재 캡처 이미지를 올리면, 과목 특성에 맞춰 시험 직전 단권화 요약과 실전 문제를 즉시 만들어 드립니다.
                </p>
              </div>

              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-2">분석할 과목 선택</label>
                <div className="grid grid-cols-2 gap-2">
                  {EXAMS.map((ex) => (
                    <button
                      key={ex.id}
                      type="button"
                      onClick={() => setAiSubject(ex.id)}
                      className={`p-2.5 rounded-xl border text-xs font-semibold text-left transition-all ${
                        aiSubject === ex.id
                          ? `${ex.badgeBg} border-current`
                          : 'bg-[#0B0D16] border-white/10 text-slate-400 hover:text-white'
                      }`}
                    >
                      {ex.shortName}
                    </button>
                  ))}
                </div>
              </div>

              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-2">원클릭 벼락치기 프롬프트</label>
                <div className="flex flex-wrap gap-1.5">
                  {[
                    {
                      subj: 'embedded',
                      label: 'ATmega128 타이머/인터럽트 빈출 C코드 문제 뽑아줘',
                      prompt: 'ATmega128 시험 대비를 위해 외부 인터럽트(EICRA, EIMSK)와 타이머/카운터0(CTC/PWM 분주비 계산)가 결합된 실전 시험 예상 C언어 빈칸 채우기 및 계산 문제 3개를 풀이과정과 함께 출제해줘.'
                    },
                    {
                      subj: 'semicon',
                      label: '전자반도체 8·9단원 오픈북 핵심 공식 총정리',
                      prompt: '전자반도체공학기초 8단원과 9단원 오픈북 시험에서 가장 자주 출제되는 핵심 공식들과 각 수식의 변수 의미, 단위 환산 시 주의할 함정 포인트를 오픈북 컨닝페이퍼 형태로 일목요연하게 정리해줘.'
                    },
                    {
                      subj: 'circuit',
                      label: '회로이론 라플라스 & 2포트 회로망 킬러 해법',
                      prompt: '회로이론 시험에서 난이도가 가장 높은 라플라스 변환(초기조건 있는 RLC 회로) 및 2포트 파라미터(Z, Y, ABCD 행렬) 문제의 단계별 기계적 풀이 알고리즘과 실전 예제 2문제를 상세 풀이와 함께 알려줘.'
                    },
                    {
                      subj: 'em',
                      label: '전자기학 정전계·정자계 경계조건 & 맥스웰 유도',
                      prompt: '전자기학 시험 단골 출제인 유전체/자성체 경계조건 유도 과정과 가우스 법칙/암페어 주회법칙/영상법을 활용한 고난도 서술형 예상 문제 2개를 완벽한 수식 유도와 함께 정리해줘.'
                    }
                  ].map((preset, idx) => (
                    <button
                      key={idx}
                      type="button"
                      onClick={() => {
                        setAiSubject(preset.subj);
                        setAiPrompt(preset.prompt);
                        callGeminiCoach(preset.prompt);
                      }}
                      className="text-left text-[11px] px-3 py-1.5 rounded-xl bg-white/5 hover:bg-white/10 border border-white/10 text-slate-200 transition-all"
                    >
                      ⚡ {preset.label}
                    </button>
                  ))}
                </div>
              </div>

              <div>
                <label className="block text-xs font-semibold text-slate-300 mb-2">
                  강의 슬라이드 / 족보 사진 첨부 (선택)
                </label>
                <div className="flex items-center gap-3">
                  <label className="flex-1 cursor-pointer flex items-center justify-center gap-2 px-4 py-3 rounded-2xl bg-[#0B0D16] hover:bg-white/5 border border-dashed border-white/20 text-xs text-slate-300 transition-all">
                    <Upload className="w-4 h-4 text-rose-400" />
                    <span>이미지 파일 선택 (PNG, JPG)</span>
                    <input type="file" accept="image/*" onChange={handleFileUpload} className="hidden" />
                  </label>
                  {aiImagePreview && (
                    <div className="relative w-12 h-12 rounded-xl overflow-hidden border border-white/20 shrink-0">
                      <img src={aiImagePreview} alt="preview" className="w-full h-full object-cover" />
                      <button
                        type="button"
                        onClick={() => {
                          setAiImageBase64(null);
                          setAiImagePreview(null);
                        }}
                        className="absolute inset-0 bg-black/60 flex items-center justify-center text-white"
                      >
                        <X className="w-4 h-4" />
                      </button>
                    </div>
                  )}
                </div>
              </div>

              <div>
                <textarea
                  rows={4}
                  value={aiPrompt}
                  onChange={(e) => setAiPrompt(e.target.value)}
                  placeholder="과거 정리해둔 시험 범위 내용이나 헷갈리는 개념, 풀리지 않는 문제를 여기에 붙여넣으세요..."
                  className="w-full p-3.5 rounded-2xl bg-[#0B0D16] border border-white/15 text-xs sm:text-sm text-white placeholder:text-slate-500 focus:outline-none focus:border-violet-500"
                />
                <button
                  type="button"
                  disabled={aiLoading}
                  onClick={() => callGeminiCoach()}
                  className="mt-3 w-full py-3 rounded-2xl bg-gradient-to-r from-rose-500 to-violet-600 hover:from-rose-600 hover:to-violet-700 disabled:opacity-50 text-white font-bold text-sm flex items-center justify-center gap-2 shadow-lg shadow-rose-500/20 transition-all"
                >
                  <Send className="w-4 h-4" />
                  <span>{aiLoading ? 'AI 전략 코치가 분석 중입니다...' : '시험 맞춤형 핵심 요약 & 문제 생성'}</span>
                </button>
              </div>
            </div>

            <div className="lg:col-span-7 rounded-3xl bg-[#11131F] border border-white/10 p-5 sm:p-6 flex flex-col min-h-[480px]">
              <div className="flex items-center justify-between pb-4 border-b border-white/10">
                <div className="flex items-center gap-2">
                  <Brain className="w-5 h-5 text-violet-400" />
                  <h4 className="text-base font-bold text-white">AI 시험 족보 & 킬러문항 브리핑 보드</h4>
                </div>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => setShowCloudModal(true)}
                    className="text-xs px-3 py-1.5 rounded-xl bg-violet-500/15 hover:bg-violet-500/25 border border-violet-500/30 text-violet-300 flex items-center gap-1.5"
                  >
                    <Key className="w-3.5 h-3.5" />
                    <span>{customApiKey ? '커스텀 API 키 적용됨' : 'GitHub 배포용 API 키 설정'}</span>
                  </button>
                  {aiResponse && (
                    <button
                      onClick={() => {
                        const el = document.createElement('textarea');
                        el.value = aiResponse;
                        document.body.appendChild(el);
                        el.select();
                        document.execCommand('copy');
                        document.body.removeChild(el);
                        showToast('분석 결과가 클립보드에 복사되었습니다!');
                      }}
                      className="text-xs px-3 py-1.5 rounded-xl bg-white/10 hover:bg-white/15 text-slate-200"
                    >
                      전체 복사
                    </button>
                  )}
                </div>
              </div>

              <div className="flex-1 mt-4 overflow-y-auto">
                {aiLoading ? (
                  <div className="h-full flex flex-col items-center justify-center text-center py-16 space-y-4">
                    <div className="w-12 h-12 rounded-2xl bg-violet-500/20 border border-violet-500/40 flex items-center justify-center animate-pulse">
                      <Sparkles className="w-6 h-6 text-violet-400" />
                    </div>
                    <div>
                      <p className="text-sm font-semibold text-white">
                        시험 출제 포인트와 킬러 문항 풀이 전략을 구조화하고 있습니다...
                      </p>
                      <p className="text-xs text-slate-400 mt-1">잠시만 기다려주세요.</p>
                    </div>
                  </div>
                ) : aiResponse ? (
                  <div className="prose prose-invert max-w-none text-xs sm:text-sm text-slate-200 whitespace-pre-wrap leading-relaxed bg-[#0B0D16] p-5 rounded-2xl border border-white/10">
                    {aiResponse}
                  </div>
                ) : (
                  <div className="h-full flex flex-col items-center justify-center text-center py-16 px-4 text-slate-400 space-y-3">
                    <FileText className="w-10 h-10 text-slate-600" />
                    <p className="text-sm font-medium text-slate-300">
                      왼쪽에서 원클릭 벼락치기 버튼을 누르거나 강의록/문제를 입력해보세요.
                    </p>
                    <p className="text-xs text-slate-500 max-w-md">
                      ATmega128 레지스터 계산, 전자반도체 8·9단원 오픈북 요약표, 회로이론·전자기학 킬러 문제 풀이를 즉시 생성합니다.
                    </p>
                  </div>
                )}
              </div>
            </div>
          </div>
        )}
      </main>

      {cheatSheetModal && (
        <div className="fixed inset-0 z-50 bg-black/75 backdrop-blur-sm flex items-center justify-center p-4">
          <div className="w-full max-w-xl rounded-3xl bg-[#121524] border border-white/15 p-6 shadow-2xl space-y-4">
            <div className="flex items-center justify-between">
              <h3 className="text-lg font-bold text-white flex items-center gap-2">
                <Bookmark className="w-5 h-5 text-amber-400" />
                <span>
                  {cheatSheetModal === 'embedded'
                    ? 'ATmega128 시험 직전 레지스터 치트시트'
                    : '전자반도체(8·9단원) 오픈북 100% 활용 전략'}
                </span>
              </h3>
              <button
                onClick={() => setCheatSheetModal(null)}
                className="p-1.5 rounded-xl bg-white/5 hover:bg-white/10 text-slate-400 hover:text-white"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="space-y-2.5 max-h-[65vh] overflow-y-auto pr-1">
              {cheatSheetModal === 'embedded'
                ? CHEAT_SHEETS.embedded.map((c, idx) => (
                    <div key={idx} className="p-3.5 rounded-2xl bg-[#0B0D16] border border-cyan-500/25">
                      <div className="text-xs font-mono font-bold text-cyan-300">{c.reg}</div>
                      <div className="text-xs text-slate-200 mt-1 leading-relaxed">{c.desc}</div>
                    </div>
                  ))
                : CHEAT_SHEETS.semicon.map((c, idx) => (
                    <div key={idx} className="p-3.5 rounded-2xl bg-[#0B0D16] border border-emerald-500/25">
                      <div className="text-xs font-bold text-emerald-300">{c.item}</div>
                      <div className="text-xs text-slate-200 mt-1 leading-relaxed">{c.detail}</div>
                    </div>
                  ))}
            </div>

            <button
              onClick={() => setCheatSheetModal(null)}
              className="w-full py-2.5 rounded-xl bg-white/10 hover:bg-white/15 text-xs font-bold text-white"
            >
              닫기
            </button>
          </div>
        </div>
      )}

      {selectedCalBlock && (
        <div
          onClick={() => setSelectedCalBlock(null)}
          className="fixed inset-0 z-50 bg-black/75 backdrop-blur-sm flex items-center justify-center p-4"
        >
          <div
            onClick={(e) => e.stopPropagation()}
            className="w-full max-w-md rounded-3xl bg-[#121524] border border-white/15 p-6 shadow-2xl space-y-4"
          >
            <div className="flex items-center justify-between">
              <span className="text-xs font-mono font-bold px-2.5 py-1 rounded-lg bg-white/10 text-slate-200">
                {selectedCalBlock.date} · {selectedCalBlock.time}
              </span>
              <button
                onClick={() => setSelectedCalBlock(null)}
                className="p-1.5 rounded-xl bg-white/5 hover:bg-white/10 text-slate-400 hover:text-white"
              >
                <X className="w-4 h-4" />
              </button>
            </div>

            <div>
              <h3 className="text-lg font-bold text-white leading-snug">
                {selectedCalBlock.title}
              </h3>
              <p className="text-xs sm:text-sm text-slate-300 mt-2 leading-relaxed bg-[#0B0D16] p-3.5 rounded-2xl border border-white/10">
                {selectedCalBlock.desc}
              </p>
            </div>

            <div className="flex items-center gap-2 pt-2">
              <button
                onClick={() => {
                  toggleBlock(selectedCalBlock.id);
                  setSelectedCalBlock(null);
                }}
                className={`flex-1 py-2.5 rounded-xl text-xs font-bold flex items-center justify-center gap-1.5 ${
                  completedBlocks[selectedCalBlock.id]
                    ? 'bg-amber-500/20 text-amber-300 border border-amber-500/40'
                    : 'bg-emerald-500 hover:bg-emerald-600 text-slate-950'
                }`}
              >
                <Check className="w-4 h-4" />
                <span>
                  {completedBlocks[selectedCalBlock.id] ? '완료 해제하기' : '이 학습 블록 완료 체크!'}
                </span>
              </button>
              <button
                onClick={() => setSelectedCalBlock(null)}
                className="px-4 py-2.5 rounded-xl bg-white/10 hover:bg-white/15 text-xs font-bold text-white"
              >
                닫기
              </button>
            </div>
          </div>
        </div>
      )}

      {showCloudModal && (
        <div
          onClick={() => setShowCloudModal(false)}
          className="fixed inset-0 z-50 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4"
        >
          <div
            onClick={(e) => e.stopPropagation()}
            className="w-full max-w-2xl rounded-3xl bg-[#121524] border border-white/15 p-6 shadow-2xl space-y-6 max-h-[90vh] overflow-y-auto"
          >
            <div className="flex items-center justify-between border-b border-white/10 pb-4">
              <div className="flex items-center gap-2.5">
                <div className="p-2.5 rounded-2xl bg-emerald-500/15 border border-emerald-500/30 text-emerald-400">
                  <Cloud className="w-5 h-5" />
                </div>
                <div>
                  <h3 className="text-lg font-bold text-white">
                    클라우드 저장소 & GitHub 배포 설정 센터
                  </h3>
                  <p className="text-xs text-slate-400">
                    실시간 자동 저장, 스마트폰·노트북 연동 코드, JSON 파일 백업 및 외부 배포용 API 키 관리
                  </p>
                </div>
              </div>
              <button
                onClick={() => setShowCloudModal(false)}
                className="p-2 rounded-xl bg-white/5 hover:bg-white/10 text-slate-400 hover:text-white"
              >
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="p-4 rounded-2xl bg-[#0B0D16] border border-emerald-500/25 space-y-3">
              <div className="flex items-center justify-between flex-wrap gap-2">
                <div className="flex items-center gap-2 text-emerald-400 text-xs font-bold">
                  <CheckCheck className="w-4 h-4" />
                  <span>1. 클라우드 실시간 자동 저장 & 기기 간 연동 코드</span>
                </div>
                <span className="text-[11px] font-mono text-slate-400">
                  {lastSavedAt ? `마지막 자동 저장: ${lastSavedAt}` : '자동 저장 대기 중'}
                </span>
              </div>

              <p className="text-xs text-slate-300 leading-relaxed">
                체크한 스케줄 블록, 과목별 단권화 체크리스트, 전자반도체 오픈북 페이지 색인은 클릭 즉시 클라우드에 자동 저장됩니다.
                스마트폰이나 아이패드에서도 똑같은 화면을 이어서 보려면 아래에 나만의 <strong>동기화 코드(Sync Code)</strong>를 입력하세요.
              </p>

              <form onSubmit={handleConnectSyncRoom} className="flex flex-col sm:flex-row gap-2 pt-1">
                <input
                  type="text"
                  value={syncRoomInput}
                  onChange={(e) => setSyncRoomInput(e.target.value)}
                  placeholder="나만의 동기화 코드 입력 (예: PASS1026)"
                  className="flex-1 px-3.5 py-2.5 rounded-xl bg-[#141829] border border-white/15 text-xs text-white placeholder:text-slate-500 focus:outline-none focus:border-emerald-400"
                />
                <button
                  type="submit"
                  className="px-4 py-2.5 rounded-xl bg-emerald-500 hover:bg-emerald-600 text-slate-950 font-bold text-xs flex items-center justify-center gap-1.5 shrink-0"
                >
                  <Link2 className="w-4 h-4" />
                  <span>동기화 코드 연결</span>
                </button>
                {activeSyncRoom && (
                  <button
                    type="button"
                    onClick={() => {
                      setActiveSyncRoom('');
                      setSyncRoomInput('');
                      showToast('개인 기본 저장소로 전환되었습니다.');
                    }}
                    className="px-3 py-2.5 rounded-xl bg-white/10 hover:bg-white/15 text-xs text-slate-300"
                  >
                    연결 해제
                  </button>
                )}
              </form>

              {user && (
                <div className="text-[11px] text-slate-500 font-mono flex items-center justify-between pt-1">
                  <span>내 클라우드 세션 ID: {user.uid}</span>
                  {activeSyncRoom && (
                    <span className="text-emerald-400 font-bold">현재 공유방: {activeSyncRoom}</span>
                  )}
                </div>
              )}
            </div>

            <div className="p-4 rounded-2xl bg-[#0B0D16] border border-cyan-500/25 space-y-3">
              <div className="flex items-center gap-2 text-cyan-400 text-xs font-bold">
                <Download className="w-4 h-4" />
                <span>2. 내 컴퓨터/스마트폰에 파일로 백업 & 불러오기 (.json)</span>
              </div>
              <p className="text-xs text-slate-300 leading-relaxed">
                GitHub Pages 등 외부 서버로 옮기거나 오프라인 상태에서 공부 기록을 소장하고 싶을 때 파일 하나로 모든 진도를 백업하고 복원할 수 있습니다.
              </p>
              <div className="flex flex-wrap gap-2.5 pt-1">
                <button
                  type="button"
                  onClick={exportBackupJson}
                  className="px-4 py-2.5 rounded-xl bg-cyan-500/20 hover:bg-cyan-500/30 border border-cyan-500/40 text-cyan-200 text-xs font-bold flex items-center gap-1.5"
                >
                  <Download className="w-4 h-4" />
                  <span>현재 진도 JSON 파일로 백업 다운로드</span>
                </button>
                <label className="cursor-pointer px-4 py-2.5 rounded-xl bg-white/10 hover:bg-white/15 border border-white/15 text-slate-200 text-xs font-bold flex items-center gap-1.5">
                  <Upload className="w-4 h-4 text-cyan-400" />
                  <span>백업 JSON 파일 불러오기</span>
                  <input type="file" accept=".json,application/json" onChange={importBackupJson} className="hidden" />
                </label>
              </div>
            </div>

            <div className="p-4 rounded-2xl bg-[#0B0D16] border border-violet-500/30 space-y-3">
              <div className="flex items-center gap-2 text-violet-400 text-xs font-bold">
                <Key className="w-4 h-4" />
                <span>3. GitHub 배포 시 AI 작동 원리 & 외부용 Gemini API 키 설정</span>
              </div>
              <div className="text-xs text-slate-300 space-y-1.5 leading-relaxed">
                <p>
                  • <strong>현재 화면(Canvas):</strong> 별도 설정 없이도 시스템이 자동으로 Gemini API 키를 주입하므로 AI가 바로 작동합니다.
                </p>
                <p>
                  • <strong>GitHub에 올릴 때 주의점:</strong> 코드 안에 API 키를 직접 적어 넣은 채로 Public GitHub 저장소에 올리면 보안 봇이 몇 초 만에 키를 감지해 차단합니다. 따라서 코드는 그대로 올리시고, 배포된 웹페이지에서 아래 입력창에 <strong>본인의 무료 Gemini API 키(Google AI Studio 발급)</strong>를 넣으면 안전하게 작동합니다!
                </p>
              </div>

              <div className="pt-1 space-y-2">
                <label className="block text-[11px] font-semibold text-slate-400">
                  외부 배포용 Gemini API Key 입력 (현재 환경에서는 비워두셔도 작동합니다)
                </label>
                <div className="flex gap-2">
                  <input
                    type="password"
                    value={customApiKey}
                    onChange={(e) => setCustomApiKey(e.target.value)}
                    placeholder="AIzaSy... (Google AI Studio에서 발급받은 키)"
                    className="flex-1 px-3.5 py-2.5 rounded-xl bg-[#141829] border border-white/15 text-xs text-white placeholder:text-slate-500 focus:outline-none focus:border-violet-400 font-mono"
                  />
                  {customApiKey && (
                    <button
                      type="button"
                      onClick={() => {
                        setCustomApiKey('');
                        showToast('커스텀 API 키가 초기화되었습니다.');
                      }}
                      className="px-3 py-2 rounded-xl bg-rose-500/20 text-rose-300 text-xs font-bold"
                    >
                      초기화
                    </button>
                  )}
                </div>
              </div>
            </div>

            <button
              onClick={() => setShowCloudModal(false)}
              className="w-full py-3 rounded-2xl bg-gradient-to-r from-rose-500 to-violet-600 text-white text-xs font-bold shadow-lg"
            >
              설정 완료 및 닫기
            </button>
          </div>
        </div>
      )}
    </div>
  );
}