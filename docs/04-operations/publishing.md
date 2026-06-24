[📚 Docs](../README.md) › [Operations](README.md) › Publishing

# Publishing — Publish lên npm

> Hướng dẫn publish package `@edupia-tutor/spec-driven-docs` lên npm: setup lần đầu, publish version mới, kiểm tra sau publish, và transfer ownership.

Package: `@edupia-tutor/spec-driven-docs`
Registry: <https://www.npmjs.com/package/@edupia-tutor/spec-driven-docs>

---

## Mục lục

1. [Yêu cầu](#1-yêu-cầu)
2. [Setup lần đầu](#2-setup-lần-đầu)
3. [Publish version mới](#3-publish-version-mới)
4. [Kiểm tra sau khi publish](#4-kiểm-tra-sau-khi-publish)
5. [Transfer package sang account khác](#5-transfer-package-sang-account-khác)

---

## 1. Yêu cầu

- Node.js đã cài (`node -v`)
- Tài khoản npm: <https://www.npmjs.com> — account có quyền publish vào scope `@edupia-tutor`

---

## 2. Setup lần đầu

### 2.1 — Đăng nhập npm

```powershell
npm login
```

Trình duyệt sẽ mở trang xác thực npm. Đăng nhập bằng account có quyền publish vào scope `@edupia-tutor`, sau đó quay lại terminal. Kiểm tra:

```powershell
npm whoami
# output: <npm-username>
```

### 2.2 — Tạo Access Token (nếu cần dùng CI hoặc tránh 2FA mỗi lần)

1. Vào <https://www.npmjs.com> → Avatar → **Access Tokens**
2. **Generate New Token** → **Granular Access Token**
3. Điền:
   - Name: `publish-spec-driven-docs`
   - Expiration: 1 year
   - **Bypass 2FA**: **bật** (bắt buộc — nếu không, publish sẽ bị chặn với lỗi `403 ... Two-factor authentication ... is required`)
   - Packages and scopes: chọn `@edupia-tutor/spec-driven-docs` (hoặc cả scope `@edupia-tutor`) → **Read and write**
4. Copy token, lưu vào nơi an toàn

> ⚠️ **KHÔNG** có flag `npm publish --token <token>` — npm sẽ hiểu `<token>` là tên package và báo lỗi `404 Not Found`. Token phải nạp qua npm config/`.npmrc` như dưới.

Publish với token — nạp token vào config trước, rồi publish:

```powershell
# Nạp token (ghi đè token đăng nhập hiện tại trong .npmrc)
npm config set //registry.npmjs.org/:_authToken <your-token>

npm publish --access public

# (tùy chọn) gỡ token khỏi .npmrc sau khi xong — sẽ phải npm login lại lần sau
npm config delete //registry.npmjs.org/:_authToken
```

**Cách khác — dùng OTP (nếu account đã bật 2FA app, không cần token bypass):**

```powershell
npm publish --access public --otp=<mã-6-số>
```

---

## 3. Publish version mới

### Bước 1 — Cập nhật nội dung commands (nếu có thay đổi)

Sửa các file trong thư mục `commands/`.

### Bước 2 — Tăng version trong `package.json` (semver)

| Loại thay đổi | Lệnh | Ví dụ |
|---|---|---|
| Fix nhỏ, sửa lỗi | `npm version patch` | 0.1.0 → 0.1.1 |
| Thêm command mới | `npm version minor` | 0.1.0 → 0.2.0 |
| Thay đổi lớn, breaking | `npm version major` | 0.1.0 → 1.0.0 |

```powershell
# Ví dụ: thêm command mới
npm version minor
```

Lệnh này tự động cập nhật `version` trong `package.json` và tạo git commit + tag.

### Bước 3 — Publish

```powershell
npm publish
```

> Nếu gặp `403 ... Two-factor authentication ... is required`: thêm `--otp=<mã-6-số>`, hoặc dùng granular token có **Bypass 2FA** (xem mục **2.2 — Tạo Access Token**). Access `public` đã set sẵn trong `package.json` (`publishConfig`).

### Bước 4 — Push git (bao gồm tag vừa tạo)

```powershell
git push && git push --tags
```

---

## 4. Kiểm tra sau khi publish

```powershell
# Xem version mới trên npm
npm view @edupia-tutor/spec-driven-docs version

# Chạy thử
npx @edupia-tutor/spec-driven-docs@latest
```

---

## 5. Transfer package sang account khác

Khi cần chuyển quyền sở hữu package cho người khác (ví dụ: sang account `edupia-tutor`):

### Thêm maintainer

```powershell
npm owner add <npm-username> @edupia-tutor/spec-driven-docs
```

### Transfer toàn bộ

```powershell
npm access grant read-write <npm-username> @edupia-tutor/spec-driven-docs
```

Hoặc làm thủ công trên website:

1. Vào <https://www.npmjs.com/package/@edupia-tutor/spec-driven-docs>
2. Tab **Settings** → **Maintainers** → thêm username mới

> **Lưu ý:** Nếu muốn đổi tên package, cần publish lại với tên mới vì npm không cho đổi tên package đã publish. Sau đó deprecate package cũ:
> ```powershell
> npm deprecate @edupia-tutor/spec-driven-docs "Moved to <new-package-name>"
> ```

---

*Xem thêm:* [Sync & Update](sync-and-update.md) (`/update-framework` kéo version mới từ npm về project) · [Bug Flow](bug-flow.md).
