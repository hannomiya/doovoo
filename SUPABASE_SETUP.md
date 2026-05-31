# Doovoo Street Market — Supabase 설정 가이드

## 순서 요약
1. SQL Editor에서 테이블 생성
2. Storage bucket 생성
3. RLS 정책 설정
4. 관리자 계정 생성

---

## STEP 1 — 테이블 생성 (SQL Editor에 붙여넣기)

Supabase 대시보드 → SQL Editor → New Query → 아래 전체 복사 후 실행

```sql
-- ────────────────────────────────────────
-- 1. products 테이블
-- ────────────────────────────────────────
create table public.products (
  id                 bigint        generated always as identity primary key,
  created_at         timestamptz   default now() not null,
  brand              text          not null,
  name               text          not null,
  category           text          not null,
  price              bigint        not null,
  description        text,
  sizes              jsonb         default '[]',
  use_box_option     boolean       default false,
  box_price          bigint        default 0,
  image_url          text,
  detail_image_urls  jsonb         default '[]'
);

-- ────────────────────────────────────────
-- 2. notices 테이블
-- ────────────────────────────────────────
create table public.notices (
  id          bigint      generated always as identity primary key,
  created_at  timestamptz default now() not null,
  title       text        not null,
  content     text        not null,
  image_url   text
);

-- ────────────────────────────────────────
-- 3. reviews 테이블
-- ────────────────────────────────────────
create table public.reviews (
  id           bigint      generated always as identity primary key,
  created_at   timestamptz default now() not null,
  author       text        default '익명',
  product_name text        default '일반 구매 상품',
  content      text        not null,
  image_url    text
);
```

---

## STEP 2 — Storage Bucket 생성

1. Supabase 대시보드 → **Storage** → **New bucket**
2. Bucket name: `doovoo-images`
3. **Public bucket** 체크 ✅ (이미지를 누구나 볼 수 있어야 하므로)
4. Create bucket

---

## STEP 3 — RLS 정책 설정 (SQL Editor)

아래를 SQL Editor에서 실행하세요.

```sql
-- ════════════════════════════════════════
-- RLS 활성화
-- ════════════════════════════════════════
alter table public.products enable row level security;
alter table public.notices  enable row level security;
alter table public.reviews  enable row level security;


-- ════════════════════════════════════════
-- products 정책
-- ════════════════════════════════════════

-- 누구나 읽기 가능 (비로그인 손님 포함)
create policy "products: 누구나 읽기"
  on public.products for select
  using (true);

-- 로그인한 관리자만 쓰기/수정/삭제
create policy "products: 관리자 insert"
  on public.products for insert
  to authenticated
  with check (true);

create policy "products: 관리자 update"
  on public.products for update
  to authenticated
  using (true);

create policy "products: 관리자 delete"
  on public.products for delete
  to authenticated
  using (true);


-- ════════════════════════════════════════
-- notices 정책
-- ════════════════════════════════════════
create policy "notices: 누구나 읽기"
  on public.notices for select
  using (true);

create policy "notices: 관리자 insert"
  on public.notices for insert
  to authenticated
  with check (true);

create policy "notices: 관리자 delete"
  on public.notices for delete
  to authenticated
  using (true);


-- ════════════════════════════════════════
-- reviews 정책
-- ════════════════════════════════════════
create policy "reviews: 누구나 읽기"
  on public.reviews for select
  using (true);

create policy "reviews: 관리자 insert"
  on public.reviews for insert
  to authenticated
  with check (true);

create policy "reviews: 관리자 delete"
  on public.reviews for delete
  to authenticated
  using (true);


-- ════════════════════════════════════════
-- Storage 정책 (doovoo-images bucket)
-- ════════════════════════════════════════

-- 누구나 이미지 열람 가능
create policy "storage: 누구나 읽기"
  on storage.objects for select
  using ( bucket_id = 'doovoo-images' );

-- 로그인한 관리자만 업로드 가능
create policy "storage: 관리자 업로드"
  on storage.objects for insert
  to authenticated
  with check ( bucket_id = 'doovoo-images' );

-- 로그인한 관리자만 삭제 가능
create policy "storage: 관리자 삭제"
  on storage.objects for delete
  to authenticated
  using ( bucket_id = 'doovoo-images' );
```

---

## STEP 4 — 관리자 계정 생성

1. Supabase 대시보드 → **Authentication** → **Users** → **Add user**
2. 이메일과 비밀번호 입력 후 생성
3. `doovoo_admin.html` 에서 해당 이메일/비밀번호로 로그인

> ⚠️ 이 계정 정보는 절대 외부에 공유하지 마세요.
> Supabase Auth는 토큰 기반이라 비밀번호가 코드에 노출되지 않습니다.

---

## 파일 구조

| 파일 | 설명 |
|---|---|
| `doovoo_shop.html` | 고객용 쇼핑몰 (읽기 전용) |
| `doovoo_admin.html` | 관리자 전용 백오피스 (로그인 필요) |

---

## 변경된 컬럼명 정리

기존 코드에서 변경된 부분입니다.

| 기존 (base64) | 변경 (Storage URL) |
|---|---|
| `image` | `image_url` |
| `detailImages` (jsonb) | `detail_image_urls` (jsonb) |
| `useBoxOption` | `use_box_option` |
| `boxPrice` | `box_price` |
| `productName` (reviews) | `product_name` |
