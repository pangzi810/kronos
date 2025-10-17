# Technical Design Document

## 概要
本設計書は、工数記録画面において承認済み日付の入力を制限する機能の技術設計を定義します。承認済みの日付に対して視覚的フィードバックを提供し、データ整合性を保証するための包括的なソリューションを実装します。

## アーキテクチャ概要

### システムコンポーネント
```
┌─────────────────────────────────────────────────────────────────────┐
│                           Frontend (Vue 3)                           │
├───────────────────────────────────────────────────────────────────────┤
│  WorkRecordsView ──> WorkHoursTable ──> ApprovalStatusBanner        │
│         │                   │                    │                   │
│         └──────────────────┼────────────────────┘                   │
│                             │                                        │
│                    developerApi.service                              │
└─────────────────────────────────────────────────────────────────────┘
                             │
                             │ HTTP/REST
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          Backend (Spring Boot)                       │
├───────────────────────────────────────────────────────────────────────┤
│   WorkRecordController ──> WorkRecordApplicationService             │
│         │                           │                                │
│         │                           ▼                                │
│         │               WorkRecordApprovalRepository                 │
│         │                           │                                │
│         └───────────────────────────┘                                │
│                                                                      │
│   Validation Layer: WorkRecordValidationService                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 詳細設計

### 1. フロントエンド実装

#### 1.1 既存API拡張による承認ステータス統合

**既存の工数取得APIレスポンスを拡張**
```typescript
// GET /api/work-records/user/{userId}/{date}
// 単一日付専用の新規エンドポイントで承認ステータスを含める

interface WorkRecordsResponse {
  // 既存のWorkRecordリスト
  workRecords: WorkRecord[];
  
  // 対象日付の承認ステータス（WorkRecordApprovalをそのまま使用）
  workRecordApproval: WorkRecordApproval | null;
}

interface WorkRecordApproval {
  userId: string;
  workDate: string;  // YYYY-MM-DD形式
  approvalStatus: 'PENDING' | 'APPROVED' | 'REJECTED';
  approverId?: string;
  approvedAt?: string;
  rejectionReason?: string;
  createdAt: string;
  updatedAt: string;
}

interface WorkRecord {
  id: string;
  userId: string;
  projectId: string;
  workDate: string;
  categoryHours: CategoryHours[];
  memo?: string;
  createdAt: string;
  updatedAt: string;
}
```

#### 1.2 WorkHoursTable.vue コンポーネント拡張

**Props追加**
```typescript
interface WorkHoursTableProps {
  // 既存props
  date: string;
  projects: Project[];
  categories: WorkCategory[];
  initialRecords: WorkRecord[];
  userId: string;
  
  // 新規props
  approvalStatus?: ApprovalStatusItem; // 承認ステータス情報
}
```

**工数記録と承認ステータスの統合管理**
```typescript
// composables/useWorkRecordsWithApproval.ts
import { ref, computed, watch } from 'vue'
import { developerApi } from '@/services/developerApi'

export function useWorkRecordsWithApproval(userId: string, selectedDate: Ref<string>) {
  const response = ref<WorkRecordsResponse | null>(null)
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  // 選択日の工数記録取得
  const currentDateRecords = computed(() => {
    if (!response.value) return []
    return response.value.workRecords.filter(r => r.workDate === selectedDate.value)
  })
  
  // 承認ステータス取得（WorkRecordApprovalを直接使用）
  const workRecordApproval = computed(() => {
    return response.value?.workRecordApproval || null
  })
  
  const isApproved = computed(() => 
    workRecordApproval.value?.approvalStatus === 'APPROVED'
  )
  
  const fetchWorkRecordsWithApproval = async () => {
    loading.value = true
    error.value = null
    try {
      // 新規の単一日付APIを使用
      const apiResponse = await developerApi.getWorkRecordsByDate(
        userId,
        selectedDate.value
      )
      response.value = apiResponse
    } catch (e) {
      error.value = 'Failed to fetch work records'
      console.error(e)
    } finally {
      loading.value = false
    }
  }
  
  // 日付変更時に自動取得
  watch(selectedDate, fetchWorkRecordsWithApproval, { immediate: true })
  
  return {
    workRecords: currentDateRecords,
    workRecordApproval,
    isApproved,
    loading,
    error,
    refresh: fetchWorkRecordsWithApproval
  }
}
```

#### 1.3 承認済みバナーコンポーネント (新規)

**components/ApprovalStatusBanner.vue**
```vue
<template>
  <v-alert
    v-if="workRecordApproval?.approvalStatus === 'APPROVED'"
    type="error"
    variant="flat"
    class="approval-banner mb-4"
    density="comfortable"
    :icon="false"
  >
    <div class="d-flex align-center justify-space-between">
      <div class="d-flex align-center">
        <v-icon size="24" class="mr-2">mdi-lock</v-icon>
        <div>
          <div class="text-h6">🔒 承認済み - 編集不可</div>
          <div class="text-caption">
            承認日時: {{ formattedApprovalDate }} | 
            承認者: {{ approverName }}
          </div>
        </div>
      </div>
      <v-tooltip location="bottom">
        <template v-slot:activator="{ props }">
          <v-icon v-bind="props" size="20">mdi-information-outline</v-icon>
        </template>
        <span>承認済みの日付は編集できません。変更が必要な場合は上司に連絡してください。</span>
      </v-tooltip>
    </div>
  </v-alert>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import type { WorkRecordApproval } from '@/types/api'

const props = defineProps<{
  workRecordApproval: WorkRecordApproval | null
}>()

const formattedApprovalDate = computed(() => {
  if (!props.workRecordApproval?.approvedAt) return ''
  const date = new Date(props.workRecordApproval.approvedAt)
  return new Intl.DateTimeFormat('ja-JP', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  }).format(date)
})
</script>

<style scoped>
.approval-banner {
  background-color: #f44336 !important;
  color: white !important;
}
</style>
```

#### 1.4 入力フィールドの無効化処理

**WorkHoursTable.vue の更新**
```vue
<template>
  <!-- 承認済みバナー表示 -->
  <ApprovalStatusBanner :approval-status="approvalStatus" />
  
  <!-- 既存のテーブル構造 -->
  <v-table fixed-header class="hours-input-table">
    <!-- ... -->
    <tbody>
      <tr v-for="project in projects" :key="project.id">
        <!-- ... -->
        <td v-for="category in categories" :key="`${project.id}-${category.id}`">
          <v-text-field
            v-model.number="hoursData[project.id][category.code.value]"
            :disabled="isApproved"
            :readonly="isApproved"
            type="number"
            min="0"
            max="24"
            step="0.5"
            density="compact"
            variant="outlined"
            hide-details
            @update:model-value="onHoursChange(project.id, category.code.value)"
            :class="{ 
              'hours-input': true,
              'hours-input--disabled': isApproved 
            }"
          >
            <!-- ... -->
          </v-text-field>
        </td>
        <!-- ... -->
      </tr>
    </tbody>
  </v-table>
  
  <!-- 保存ボタンの無効化 -->
  <v-btn
    color="primary"
    @click="saveAllRecords"
    :disabled="!hasChanges || saving || isApproved"
    :loading="saving"
  >
    <v-icon start>mdi-content-save</v-icon>
    {{ isApproved ? 'Read Only' : 'Save' }}
  </v-btn>
</template>

<style scoped>
/* 承認済み入力フィールドのスタイル */
.hours-input--disabled :deep(.v-field) {
  background-color: #f5f5f5 !important;
  cursor: not-allowed !important;
}

.hours-input--disabled :deep(input) {
  cursor: not-allowed !important;
}
</style>
```

### 2. バックエンド実装

#### 2.1 既存APIの拡張による承認ステータス統合

**WorkRecordsResponse.java (新規レスポンスDTO)**
```java
package com.devhour.application.dto;

import com.devhour.domain.model.entity.WorkRecord;
import com.devhour.domain.model.entity.WorkRecordApproval;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class WorkRecordsResponse {
    // 既存のWorkRecordリスト
    private List<WorkRecord> workRecords;
    
    // 対象日付の承認ステータス（WorkRecordApprovalエンティティを直接使用）
    private WorkRecordApproval workRecordApproval;
}
```

**WorkRecordApplicationService.java に新規メソッド追加**
```java
/**
 * 単一日付の工数記録と承認ステータスを取得
 * 
 * @param userId ユーザーID
 * @param date 対象日付
 * @return 工数記録と承認ステータスを含むレスポンス
 */
@Transactional(readOnly = true)
public WorkRecordsResponse getWorkRecordsWithApprovalStatus(String userId, LocalDate date) {
    // 工数記録取得
    List<WorkRecord> workRecords = findByUserIdAndDate(userId, date);
    
    // 承認ステータス取得（存在しない場合は新規作成）
    WorkRecordApproval approval = workRecordApprovalRepository
        .findByUserIdAndDate(userId, date)
        .orElse(new WorkRecordApproval(userId, date));
    
    // レスポンスオブジェクト構築
    return WorkRecordsResponse.builder()
        .workRecords(workRecords)
        .workRecordApproval(approval)
        .build();
}
```

**WorkRecordController.java のエンドポイント実装**
```java
/**
 * 単一日付の工数記録と承認ステータス取得
 */
@GetMapping("/user/{userId}/{date}")
@Operation(summary = "単一日付の工数記録取得",
          description = "指定日の工数記録と承認ステータスを取得します")
public ResponseEntity<WorkRecordsResponse> getWorkRecordsByDate(
        @PathVariable String userId,
        @PathVariable @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate date,
        @AuthenticationPrincipal CustomUserDetails userDetails) {
    
    // アプリケーションサービスに処理を委譲
    WorkRecordsResponse response = 
        workRecordApplicationService.getWorkRecordsWithApprovalStatus(userId, date);
    
    return ResponseEntity.ok(response);
}

/**
 * 既存の期間指定エンドポイント（変更なし）
 */
@GetMapping("/user/{userId}/period")
@Operation(summary = "期間指定で工数記録検索",
          description = "指定期間の工数記録を取得します")
public ResponseEntity<List<WorkRecord>> getWorkRecordsByPeriod(
        @PathVariable String userId,
        @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate startDate,
        @RequestParam @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate endDate,
        @AuthenticationPrincipal CustomUserDetails userDetails) {
    
    List<WorkRecord> workRecords = 
        workRecordApplicationService.findByUserIdAndDateRange(userId, startDate, endDate);
    
    return ResponseEntity.ok(workRecords);
}
```



### 3. エラーハンドリング

#### 3.1 フロントエンドエラー処理

```typescript
// components/WorkHoursTable.vue
const handleApprovalError = (error: any) => {
  if (error.response?.status === 403) {
    // 承認済みエラー
    toast.error('この日付は承認済みのため編集できません')
    
    // 承認ステータスを再取得
    refreshApprovalStatus()
  } else {
    // その他のエラー
    toast.error('エラーが発生しました')
  }
}
```

#### 3.2 リトライメカニズム

```typescript
// utils/retryable.ts
export async function retryable<T>(
  fn: () => Promise<T>,
  maxAttempts = 3,
  delay = 1000
): Promise<T> {
  let lastError: any
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn()
    } catch (error) {
      lastError = error
      
      if (attempt < maxAttempts) {
        await new Promise(resolve => setTimeout(resolve, delay * attempt))
      }
    }
  }
  
  throw lastError
}
```

## テスト戦略

### 1. ユニットテスト

#### フロントエンド
```typescript
// WorkHoursTable.spec.ts
describe('WorkHoursTable', () => {
  it('承認済み日付の場合、入力フィールドが無効化される', async () => {
    const wrapper = mount(WorkHoursTable, {
      props: {
        approvalStatus: {
          approvalStatus: 'APPROVED',
          workDate: '2024-01-15',
          approverId: 'supervisor1',
          approverName: '上司太郎',
          approvedAt: '2024-01-16T10:00:00'
        }
      }
    })
    
    const inputs = wrapper.findAll('input[type="number"]')
    inputs.forEach(input => {
      expect(input.attributes('disabled')).toBeDefined()
    })
  })
})
```

#### バックエンド
```java
@Test
void 承認済み日付の更新は拒否される() {
    // Given
    WorkRecordApproval approval = new WorkRecordApproval("user1", LocalDate.now());
    approval.approve("supervisor1");
    workRecordApprovalRepository.save(approval);
    
    WorkRecordCreateRequest request = new WorkRecordCreateRequest();
    request.setUserId("user1");
    request.setWorkDate(LocalDate.now());
    
    // When/Then
    assertThrows(BusinessException.class, () -> 
        workRecordApplicationService.createOrUpdateWorkRecord(request)
    );
}
```

### 2. 統合テスト

```java
@SpringBootTest
@AutoConfigureMockMvc
class ApprovalRestrictionIntegrationTest {
    
    @Test
    void 承認済み日付の入力制限が機能する() throws Exception {
        // 承認済みデータを準備
        // ...
        
        // 更新リクエスト
        mockMvc.perform(post("/api/work-records")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isForbidden())
                .andExpect(jsonPath("$.message")
                    .value("承認済みの日付のデータは変更できません"));
    }
}
```

## 実装の要件マッピング

| 要件ID | 実装内容 | 検証方法 |
|--------|----------|----------|
| Req 1.1-1.5 | WorkHoursTable.vueの入力制限ロジック | ユニットテスト |
| Req 2.1-2.5 | CSSスタイリングとUI状態管理 | ビジュアルテスト |
| Req 2.6-2.8 | ApprovalStatusBannerコンポーネント | コンポーネントテスト |
| Req 3.1-3.5 | 承認ステータス取得API | APIテスト |
| Req 4.1-4.6 | ツールチップとトースト通知 | E2Eテスト |
| Req 5.1-5.5 | パフォーマンス最適化 | パフォーマンステスト |
| Req 6.1-6.5 | バックエンド検証ロジック | 統合テスト |

## デプロイメント考慮事項

### データベースマイグレーション
既存のWorkRecordApprovalテーブルを使用するため、追加のマイグレーションは不要。

### モニタリング
- 承認ステータス取得APIのレスポンスタイム監視
- 403エラー（承認済み更新試行）の頻度監視

## セキュリティ考慮事項

1. **認証・認可**: JWT認証により自分の承認ステータスのみ取得可能
2. **データ検証**: フロントエンドとバックエンドの二重検証
3. **監査ログ**: 承認済みデータへの更新試行を記録
4. **CSRF対策**: Spring Securityのデフォルト設定を使用
5. **SQLインジェクション対策**: MyBatisのパラメータバインディング使用