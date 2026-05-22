# Catatan Revisi Figma Register Pinjaman DN/LN
## Implementasi Lengkap dengan KEU-UI Design System

**Status:** ✅ READY FOR IMPLEMENTATION  
**Version:** 1.0  
**Date:** 2026-05-22  
**Author:** Copilot Design Review

---

## 📌 EXECUTIVE SUMMARY

### Problem Utama Yang Harus Diperbaiki

| # | Problem | Impact | Priority |
|---|---------|--------|----------|
| 1 | Wording terlalu teknis (ReqDoc, parent, child) | User confusion | 🔴 P1 |
| 2 | Create flow campur ReqDoc + Register | Wrong workflow | 🔴 P1 |
| 3 | Form Register belum DMFAS-first | Duplicate entry | 🔴 P1 |
| 4 | Field ReqDoc kurang lengkap | Incomplete data | 🟡 P2 |
| 5 | Status ReqDoc vs Register tercampur | Ambiguous state | 🟡 P2 |

---

## 🎯 REKOMENDASI PRIORITAS 1 (URGENT - SPRINT BERIKUTNYA)

### 1.1 NATURALISASI SEMUA WORDING

#### ❌ SEKARANG vs ✅ SESUDAH

**List Page Title:**
```
❌ "Daftar ReqDoc Pinjaman Dalam Negeri"
✅ "Daftar Dokumen Permintaan Pinjaman Dalam Negeri"
```

**Create Button:**
```
❌ "Catat ReqDoc Baru"
✅ "Catat Dokumen Permintaan Baru"
```

**Table Columns:**
```
❌ "No. ReqDoc"          →  ✅ "No. Dok. Permintaan"
❌ "Child Register"      →  ✅ "Jumlah Register"
❌ "Kem. SPV"            →  ✅ "Status Perbaikan"
```

**Helper Text:**
```
❌ "Hierarki data: ReqDoc adalah dokumen permintaan pencatatan pinjaman 
   (parent). Setiap baris = satu ReqDoc (parent). Child register adalah 
   daftar register yang melekat pada satu dokumen permintaan."

✅ [HILANGKAN SEPENUHNYA]
   Atau ganti dengan: "Satu dokumen dapat memuat lebih dari satu register."
```

**Implementation in React:**
```tsx
// components/ListDocuments.tsx
import { KeuPageHeader, KeuTable, KeuButton } from 'keu-ui';

export const ListDocumentsPage = () => {
  return (
    <>
      <KeuPageHeader 
        title="Daftar Dokumen Permintaan Pinjaman Dalam Negeri"
        action={
          <KeuButton 
            label="Catat Dokumen Permintaan Baru"
            onClick={handleCreate}
          />
        }
      />

      <KeuTable
        columns={[
          { key: 'no', label: 'No. Dok. Permintaan' },
          { key: 'date', label: 'Tanggal Permintaan' },
          { key: 'subject', label: 'Perihal' },
          { key: 'count', label: 'Jumlah Register' },
          { key: 'status', label: 'Status Perbaikan' },
        ]}
        data={documents}
      />
    </>
  );
};
```

---

### 1.2 SPLIT CREATE FLOW (REQDOC ONLY FIRST)

#### User Journey: BEFORE (Salah) → AFTER (Benar)

**BEFORE (❌ CAMPUR):**
```
1. Klik "Catat ReqDoc Baru"
   ↓
2. Form: ReqDoc + Register Pertama sekaligus
   - Field ReqDoc (nomor, tanggal, dll)
   - Field Register (DMFAS ID, reference, dll)
   - Select: Tipe Register (Baru/Addendum/Pembatalan)
   ↓
3. Simpan → Keluar (CONFUSION!)
```

**AFTER (✅ TERPISAH):**
```
1. Klik "Catat Dokumen Permintaan Baru"
   ↓
2. FORM 1: Dokumen Permintaan (SAJA)
   ┌─────────────────────────────────┐
   │ IDENTITAS DOKUMEN               │
   │ - No. Dok (auto)                │
   │ - Tanggal Permintaan            │
   │ - Perihal/Subject               │
   │ - Mekanisme                     │
   │ - Jenis Hutang                  │
   │                                 │
   │ PENGIRIM & KONTAK               │
   │ - Nama Pengirim                 │
   │ - Jabatan                       │
   │ - Kementerian/Lembaga (K/L)     │
   │ - Satuan Kerja (Satker)         │
   │ - Alamat                        │
   │ - Telepon, Fax                  │
   │ - Email 1, Email 2              │
   │                                 │
   │ CATATAN                         │
   │ - Catatan Dokumen               │
   │                                 │
   │ [Batal] [Simpan & Lanjut]       │
   └─────────────────────────────────┘
   ↓
3. ReqDoc Created ✓ (ID: DN-2025-001)
   ↓
4. Redirect → Detail Dokumen Permintaan
   ┌─────────────────────────────────┐
   │ Tab: [Dokumen] [Register] [PDF] │
   │                                 │
   │ Tab Register (KOSONG):          │
   │ [+ Tambah Register Baru]        │
   │ [+ Tambah Addendum]             │
   │ [+ Tambah Pembatalan]           │
   │                                 │
   │ Tabel: (no data)                │
   └─────────────────────────────────┘
   ↓
5. Klik: "Tambah Register Baru"
   ↓
6. FORM 2: Register (dengan DMFAS)
   [STEP 1] DMFAS ID Lookup
   [STEP 2] Auto-Populate dari DMFAS
   [STEP 3] User Validate
   ↓
7. Register Created ✓
   ↓
8. Back to Detail → Register Appear
```

**React Implementation:**

```tsx
// pages/RegisterLoan/CreateReqDoc.tsx
import React, { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import {
  KeuForm,
  KeuFormGroup,
  KeuInput,
  KeuSelect,
  KeuDatePicker,
  KeuTextArea,
  KeuButton,
} from 'keu-ui';
import { reqdocService } from '../../services/reqdocService';

export const CreateReqDocPage: React.FC = () => {
  const navigate = useNavigate();
  const [loading, setLoading] = useState(false);

  const handleSubmit = async (formData) => {
    setLoading(true);
    try {
      const reqdoc = await reqdocService.createReqDoc(formData);
      // Redirect to detail page
      navigate(`/register/dn/detail/${reqdoc.rd_id}`);
    } catch (error) {
      console.error('Error creating ReqDoc:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <KeuForm onSubmit={handleSubmit} loading={loading}>
      {/* SECTION 1: Identitas Dokumen */}
      <KeuFormGroup title="Identitas Dokumen">
        <KeuInput 
          label="No. Dok. Permintaan" 
          disabled 
          value="(Auto-generated)"
        />
        <KeuDatePicker 
          label="Tanggal Permintaan" 
          required 
        />
        <KeuInput 
          label="Perihal/Subject" 
          required 
        />
        <KeuSelect 
          label="Mekanisme" 
          required
          options={[
            { value: 'bilateral', label: 'Bilateral Loan' },
            { value: 'multilateral', label: 'Multilateral Loan' },
          ]}
        />
        <KeuSelect 
          label="Jenis Hutang" 
          required
          options={[
            { value: 'concessional', label: 'Concessional' },
            { value: 'non_concessional', label: 'Non-Concessional' },
          ]}
        />
      </KeuFormGroup>

      {/* SECTION 2: Pengirim & Kontak */}
      <KeuFormGroup title="Pengirim & Kontak">
        <KeuInput 
          label="Nama Pengirim" 
          required 
        />
        <KeuInput 
          label="Jabatan" 
          required 
        />
        <KeuSelect 
          label="Kementerian/Lembaga (K/L)" 
          required
          searchable
          options={[]} // Will be populated from API
        />
        <KeuSelect 
          label="Satuan Kerja (Satker)" 
          required
          searchable
          options={[]} // Will be populated based on K/L selection
        />
        <KeuTextArea 
          label="Alamat" 
          required 
        />
        <KeuInput 
          label="Telepon" 
          type="tel" 
          required 
        />
        <KeuInput 
          label="Fax" 
          type="tel" 
        />
        <KeuInput 
          label="Email 1" 
          type="email" 
          required 
        />
        <KeuInput 
          label="Email 2" 
          type="email" 
        />
      </KeuFormGroup>

      {/* SECTION 3: Catatan */}
      <KeuFormGroup title="Catatan">
        <KeuTextArea 
          label="Catatan Dokumen" 
          placeholder="Masukkan catatan tambahan untuk dokumen ini..."
        />
      </KeuFormGroup>

      {/* ACTION BUTTONS */}
      <div className="form-actions" style={{ marginTop: '24px' }}>
        <KeuButton 
          type="reset" 
          label="Batal" 
          variant="secondary"
        />
        <KeuButton 
          type="submit" 
          label="Simpan & Lanjut ke Register"
        />
      </div>
    </KeuForm>
  );
};
```

---

### 1.3 IMPLEMENT DMFAS-FIRST REGISTER FORM

#### Flow Diagram: DMFAS-First Pattern

```
┌─────────────────────────────────────────────────┐
│ STEP 1: PILIH DMFAS ID                          │
├─────────────────────────────────────────────────┤
│                                                 │
│ Field: DMFAS ID / Reference (Searchable)       │
│ [Search: "project", "LN-2025", "12345"]        │
│                                                 │
│ Results:                                        │
│ • 12345 - Project Alpha (LN-2025-001)          │
│ • 12346 - Project Beta (LN-2025-002)           │
│                                                 │
│ [User selects one]                             │
└─────────────┬───────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────┐
│ STEP 2: SYSTEM AUTO-POPULATE                    │
├─────────────────────────────────────────────────┤
│                                                 │
│ [Loading] Mengambil data dari DMFAS...         │
│                                                 │
│ API Call: GET /dmfas/loans/12345                │
│ Response received ✓                             │
│                                                 │
└─────────────┬───────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────────┐
│ STEP 3: REVIEW & VALIDATE DATA                  │
├─────────────────────────────────────────────────┤
│                                                 │
│ ✓ Data dari DMFAS (READ-ONLY):                 │
│   DMFAS ID:           [12345]                  │
│   Reference:          [LN-2025-001]            │
│   Other Reference:    [PN-123456]              │
│   Short Name:         [Project Alpha]          │
│   Long Name:          [Project Alpha Dev]      │
│   Instrument Status:  [Active]                 │
│   Signed Date:        [2025-01-15]             │
│   Effective Date:     [2025-02-01]             │
│   Effective Limit:    [2025-12-31]             │
│   Drawing Limit:      [2025-12-31]             │
│   Currency:           [USD]                    │
│   Amount:             [50,000,000]             │
│   Donor:              [ADB]                    │
│   Beneficiary:        [Ministry of XYZ]        │
│   Main Beneficiary:   [Department ABC]         │
│                                                 │
│ ⚙️  Field Lokal (EDITABLE):                    │
│   Catatan Register:   [textarea]               │
│   Status Lokal:       [dropdown]               │
│   Officer Responsible: [text]                  │
│                                                 │
└─────────────┬───────────────────────────────────┘
              ↓
          [Batal] [Simpan]
              ↓
        Register Created ✓
        Linked to ReqDoc ✓
```

**React Implementation:**

```tsx
// pages/RegisterLoan/CreateRegister.tsx
import React, { useState, useEffect } from 'react';
import {
  KeuForm,
  KeuFormGroup,
  KeuSelect,
  KeuInput,
  KeuTextArea,
  KeuButton,
  KeuAlert,
  KeuSkeleton,
} from 'keu-ui';
import { dmfasService } from '../../services/dmfasService';
import { registerService } from '../../services/registerService';

interface CreateRegisterProps {
  reqdocId: string;
  onSuccess: () => void;
}

export const CreateRegisterForm: React.FC<CreateRegisterProps> = ({
  reqdocId,
  onSuccess,
}) => {
  const [selectedDmfasId, setSelectedDmfasId] = useState<string | null>(null);
  const [dmfasData, setDmfasData] = useState<any>(null);
  const [loading, setLoading] = useState(false);
  const [searchResults, setSearchResults] = useState<any[]>([]);
  const [localFields, setLocalFields] = useState({
    remark: '',
    localStatus: '',
    officerResponsible: '',
  });

  // Handle DMFAS search
  const handleDmfasSearch = async (query: string) => {
    if (!query) {
      setSearchResults([]);
      return;
    }
    try {
      const results = await dmfasService.searchLoans(query);
      setSearchResults(results);
    } catch (error) {
      console.error('Error searching DMFAS:', error);
    }
  };

  // Handle DMFAS selection
  const handleDmfasSelect = async (dmfasId: string) => {
    setSelectedDmfasId(dmfasId);
    setLoading(true);
    try {
      const data = await dmfasService.getLoanDetails(dmfasId);
      setDmfasData(data);
    } catch (error) {
      console.error('Error fetching DMFAS details:', error);
    } finally {
      setLoading(false);
    }
  };

  // Handle form submit
  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!selectedDmfasId || !dmfasData) return;

    try {
      const registerData = {
        reqdoc_id: reqdocId,
        dmfas_id: selectedDmfasId,
        // DMFAS fields (auto-populated)
        reference: dmfasData.reference,
        other_reference: dmfasData.otherRef,
        short_name: dmfasData.shortName,
        long_name: dmfasData.longName,
        instrument_status: dmfasData.status,
        signed_date: dmfasData.signedDate,
        effective_date: dmfasData.effectiveDate,
        currency: dmfasData.currency,
        amount: dmfasData.amount,
        donor: dmfasData.donor,
        beneficiary: dmfasData.beneficiary,
        main_beneficiary: dmfasData.mainBeneficiary,
        // Local fields (user-editable)
        remark: localFields.remark,
        local_status: localFields.localStatus,
        officer_responsible: localFields.officerResponsible,
      };

      await registerService.createRegister(registerData);
      onSuccess();
    } catch (error) {
      console.error('Error creating register:', error);
    }
  };

  return (
    <KeuForm onSubmit={handleSubmit}>
      {/* STEP 1: DMFAS ID SELECTION */}
      <KeuFormGroup title="Pilih Data dari DMFAS">
        <KeuSelect
          label="DMFAS ID / Reference Number"
          placeholder="Cari loan dalam DMFAS..."
          searchable
          onSearch={(query) => handleDmfasSearch(query)}
          options={searchResults.map(item => ({
            value: item.id,
            label: `${item.id} - ${item.shortName} (${item.reference})`,
          }))}
          onChange={(val) => handleDmfasSelect(val)}
          required
        />
      </KeuFormGroup>

      {/* STEP 2 & 3: AUTO-POPULATED DATA + LOCAL EDITS */}
      {loading && <KeuSkeleton variant="form" />}

      {dmfasData && (
        <>
          <KeuAlert
            type="info"
            message="Berikut adalah data dari DMFAS. Sebagian besar field tidak dapat diubah. Anda hanya dapat mengedit catatan dan field lokal."
          />

          {/* READ-ONLY DMFAS FIELDS */}
          <KeuFormGroup title="Data dari DMFAS (Tidak dapat diubah)">
            <KeuInput
              label="DMFAS ID"
              value={dmfasData.id}
              disabled
            />
            <KeuInput
              label="Reference"
              value={dmfasData.reference}
              disabled
            />
            <KeuInput
              label="Other Reference"
              value={dmfasData.otherRef}
              disabled
            />
            <KeuInput
              label="Short Name"
              value={dmfasData.shortName}
              disabled
            />
            <KeuInput
              label="Long Name"
              value={dmfasData.longName}
              disabled
            />
            <KeuInput
              label="Instrument Status"
              value={dmfasData.status}
              disabled
            />
            <KeuInput
              label="Signed Date"
              type="date"
              value={dmfasData.signedDate}
              disabled
            />
            <KeuInput
              label="Effective Date"
              type="date"
              value={dmfasData.effectiveDate}
              disabled
            />
            <KeuInput
              label="Effective Limit"
              type="date"
              value={dmfasData.effectiveLimit}
              disabled
            />
            <KeuInput
              label="Drawing Limit"
              type="date"
              value={dmfasData.drawingLimit}
              disabled
            />
            <KeuInput
              label="Currency"
              value={dmfasData.currency}
              disabled
            />
            <KeuInput
              label="Amount"
              value={dmfasData.amount?.toLocaleString()}
              disabled
            />
            <KeuInput
              label="Donor"
              value={dmfasData.donor}
              disabled
            />
            <KeuInput
              label="Beneficiary"
              value={dmfasData.beneficiary}
              disabled
            />
            <KeuInput
              label="Main Beneficiary"
              value={dmfasData.mainBeneficiary}
              disabled
            />
          </KeuFormGroup>

          {/* EDITABLE LOCAL FIELDS */}
          <KeuFormGroup title="Field Lokal (Dapat diubah)">
            <KeuTextArea
              label="Catatan Register"
              value={localFields.remark}
              onChange={(val) =>
                setLocalFields({ ...localFields, remark: val })
              }
              placeholder="Masukkan catatan tambahan untuk register ini..."
            />
            <KeuSelect
              label="Status Lokal"
              options={[
                { value: 'draft', label: 'Draft' },
                { value: 'submitted', label: 'Submitted' },
                { value: 'processing', label: 'Processing' },
              ]}
              onChange={(val) =>
                setLocalFields({ ...localFields, localStatus: val })
              }
            />
            <KeuInput
              label="Officer Penanggung Jawab"
              value={localFields.officerResponsible}
              onChange={(val) =>
                setLocalFields({ ...localFields, officerResponsible: val })
              }
            />
          </KeuFormGroup>
        </>
      )}

      {/* ACTION BUTTONS */}
      <div className="form-actions" style={{ marginTop: '24px' }}>
        <KeuButton
          type="reset"
          label="Batal"
          variant="secondary"
          disabled={loading}
        />
        <KeuButton
          type="submit"
          label="Simpan Register"
          disabled={!selectedDmfasId || !dmfasData || loading}
        />
      </div>
    </KeuForm>
  );
};
```

---

## 🎯 REKOMENDASI PRIORITAS 2 (HIGH - SPRINT 2-3)

### 2.1 Complete ReqDoc Fields

**Database Migration:**
```sql
ALTER TABLE REQ_DOCS_CSO ADD (
  SENDER_POSITION VARCHAR2(100),
  SENDER_ADDRESS VARCHAR2(500),
  SENDER_PHONE VARCHAR2(20),
  SENDER_FAX VARCHAR2(20),
  SENDER_EMAIL_1 VARCHAR2(100),
  SENDER_EMAIL_2 VARCHAR2(100),
  DOC_STATUS VARCHAR2(20) DEFAULT 'draft'
);
```

### 2.2 ReqDoc Status Lifecycle

**Status Flow:**
```
Draft (default) 
  ↓ [User submit]
Submitted 
  ↓ [SPV/DJPPR start validation]
OnProcess 
  ↓ [All registers reach final state]
Completed
```

### 2.3 Complete Terminology Map

```
OLD TERMINOLOGY    →    NEW TERMINOLOGY
────────────────────────────────────────
ReqDoc             →    Dokumen Permintaan
ReqDoc Baru        →    Dokumen Permintaan Baru
No. ReqDoc         →    No. Dok. Permintaan
Child Register     →    Register Terkait
Register Count     →    Jumlah Register
Detail ReqDoc      →    Detail Dokumen Permintaan
Ringkasan Child    →    Daftar Register
Kem. SPV           →    Dikembalikan SPV
SPV Confir.        →    Dikonfirmasi SPV
On Process         →    Sedang Diproses
```

---

## ✅ IMPLEMENTATION CHECKLIST

### PHASE 1: SPRINT BERIKUTNYA (URGENT)
- [ ] Split create flow: ReqDoc only first
- [ ] Update all wording on list page
- [ ] Implement DMFAS-first register form
- [ ] Update button labels & titles everywhere
- [ ] Remove technical help text from UI

### PHASE 2: SPRINT 2-3 (HIGH)
- [ ] Add complete ReqDoc fields to form & DB
- [ ] Implement ReqDoc status lifecycle
- [ ] Create ReqDoc stepper component
- [ ] Update all terminology across UI
- [ ] Add missing API endpoints

### PHASE 3: SPRINT 3+ (NICE-TO-HAVE)
- [ ] Format K/L & Satker display
- [ ] Comprehensive testing (unit, integration, e2e)
- [ ] Accessibility audit
- [ ] Update documentation & Storybook

---

## 📦 FILES TO CREATE/MODIFY

```
CREATE:
  ✨ src/pages/RegisterLoan/CreateReqDoc.tsx
  ✨ src/components/ReqDocStepper.tsx
  ✨ src/utils/formatters.ts

MODIFY:
  📝 src/pages/RegisterLoan/index.tsx
  📝 src/pages/RegisterLoan/DetailReqDoc.tsx
  📝 src/pages/RegisterLoan/CreateRegister.tsx
  📝 src/services/reqdocService.ts
  📝 src/services/dmfasService.ts
  📝 Database schema (add DOC_STATUS)
```

---

## 🔌 REQUIRED API ENDPOINTS

```
ReqDoc:
  POST   /api/reqdoc
  GET    /api/reqdoc/:id
  GET    /api/reqdoc
  PATCH  /api/reqdoc/:id
  PATCH  /api/reqdoc/:id/status

Register:
  POST   /api/register
  GET    /api/register/:id
  GET    /api/reqdoc/:id/registers

DMFAS:
  GET    /api/dmfas/loans (search)
  GET    /api/dmfas/loans/:id

Masters:
  GET    /api/masters/ministries
  GET    /api/masters/satkers
  GET    /api/masters/mechanisms
  GET    /api/masters/debt-types
```

---

## 🎯 SUCCESS CRITERIA

✅ No technical terms (ReqDoc, parent, child) visible to users  
✅ Create ReqDoc & Register are separate flows  
✅ Register form uses DMFAS-first pattern  
✅ ReqDoc has 15+ fields matching legacy system  
✅ ReqDoc & Register status are clearly separated  
✅ All pages use KEU-UI components  
✅ Full WCAG 2.1 AA accessibility compliance  
✅ Documentation & Storybook updated  

---

**Version:** 1.0  
**Status:** Ready for Sprint Planning  
**Date:** 2026-05-22
