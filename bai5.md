# Bài 5: Bài Tập Sáng Tạo - Xây Dựng Workflow Tích Hợp

## 1. Thiết kế luồng (What & Why)

### Input (Dữ liệu đầu vào)
| Tham số | Kiểu dữ liệu | Mô tả |
|---------|---------------|-------|
| `orderAmount` | `double` | Giá trị đơn hàng (VNĐ). Ví dụ: 350000.0 |
| `isWeekend` | `boolean` | `true` nếu mua vào Thứ 7 hoặc Chủ Nhật |
| `isVip` | `boolean` | `true` nếu khách hàng thuộc hạng VIP |

### Output (Kết quả trả về)
| Tham số | Kiểu dữ liệu | Mô tả |
|---------|---------------|-------|
| `totalLoyaltyPoints` | `int` | Tổng điểm thưởng (số nguyên, >= 0) |

### Sơ đồ xử lý logic (ASCII)

```text
                      ┌─────────────────────────┐
                      │      INPUT DATA          │
                      │ orderAmount, isWeekend,   │
                      │ isVip                     │
                      └────────────┬──────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │  VALIDATE: orderAmount   │
                      │  <= 0 hoặc < 100.000?    │
                      └────────────┬──────────────┘
                            YES /     \ NO
                               /       \
                              ▼         ▼
                    ┌──────────┐  ┌──────────────────────┐
                    │ Return 0 │  │ BƯỚC 1: Tính điểm    │
                    │ points   │  │ cơ sở (Base Points)  │
                    └──────────┘  │ = orderAmount/100000 │
                                  │ (Lấy phần nguyên)    │
                                  └──────────┬───────────┘
                                             │
                                             ▼
                                  ┌──────────────────────┐
                                  │ BƯỚC 2: Cuối tuần?   │
                                  │ isWeekend == true?    │
                                  │ → Điểm = Điểm × 2   │
                                  └──────────┬───────────┘
                                             │
                                             ▼
                                  ┌──────────────────────┐
                                  │ BƯỚC 3: VIP?         │
                                  │ isVip == true?       │
                                  │ → Điểm = Điểm × 1.2 │
                                  │ (Làm tròn lên ceil)  │
                                  └──────────┬───────────┘
                                             │
                                             ▼
                                  ┌──────────────────────┐
                                  │  OUTPUT:             │
                                  │  totalLoyaltyPoints  │
                                  └──────────────────────┘
```

**Lý do thiết kế:** Tách bạch logic thành 3 bước tuần tự đảm bảo đúng thứ tự ưu tiên theo yêu cầu Business Analyst:
1. Tính điểm cơ sở trước.
2. Áp dụng ưu đãi cuối tuần (nhân đôi).
3. Cuối cùng mới áp dụng ưu đãi VIP (+20%) — vì đề bài nói rõ "sau khi đã tính các ưu đãi khác".

---

## 2. Workflow / Mega-Prompt

```
Đóng vai trò là AI Solution Architect & Senior Java Developer cho hệ thống TechShop.

Nhiệm vụ: Triển khai tính năng "Tính điểm thưởng (Loyalty Points)" đạt chuẩn production,
bao gồm: mã nguồn Java có Javadoc + Unit Test JUnit 5.

Nghiệp vụ (Business Rules) từ Business Analyst:
1. Cứ tiêu 100.000 VNĐ thì được 1 điểm thưởng (tính phần nguyên, VD: 150k → 1 điểm).
2. Nếu mua hàng vào cuối tuần (Thứ 7 hoặc Chủ Nhật) → nhân đôi tổng số điểm.
3. Khách hàng hạng VIP → cộng thêm 20% vào số điểm cuối cùng (sau khi đã tính ưu đãi cuối tuần).
   Kết quả làm tròn lên (Math.ceil) thành số nguyên.
4. Edge cases cần xử lý: đơn hàng có giá trị âm, bằng 0, hoặc < 100.000 VNĐ → điểm = 0.

Yêu cầu cụ thể:

Bước 1 - Sinh class LoyaltyPointCalculator:
- Phương thức: calculatePoints(double orderAmount, boolean isWeekend, boolean isVip)
- Trả về int (tổng điểm thưởng).
- Kèm đầy đủ Javadoc cho class và method (@param, @return).
- Thêm inline comment giải thích từng bước tính toán.

Bước 2 - Sinh class LoyaltyPointCalculatorTest (JUnit 5):
Tối thiểu 5 test case bao phủ các kịch bản sau:
- Mua hàng ngày thường, không phải VIP (kiểm tra điểm cơ sở).
- Mua hàng cuối tuần, không phải VIP (kiểm tra nhân đôi).
- Mua hàng ngày thường, là VIP (kiểm tra cộng 20% và làm tròn lên).
- Mua hàng cuối tuần + VIP (kiểm tra thứ tự áp dụng ưu đãi).
- Edge cases: giá trị đơn hàng âm, bằng 0, số lẻ < 100.000 VNĐ.
Mỗi test case phải có comment giải thích phép tính kỳ vọng.

Định dạng: Trả về 2 khối code block Java. Không giải thích thêm ngoài code.
```

---

## 3. Text Log Hội Thoại Với AI (Kiểm chứng thực tế)

### Prompt gửi cho AI:

> Đóng vai trò là AI Solution Architect & Senior Java Developer cho hệ thống TechShop.
> Nhiệm vụ: Triển khai tính năng "Tính điểm thưởng (Loyalty Points)" đạt chuẩn production...
> [Toàn bộ nội dung Mega-Prompt ở mục 2 phía trên]

### Kết quả AI trả về:

**Class chính — LoyaltyPointCalculator.java:**

```java
/**
 * Lớp chịu trách nhiệm tính toán điểm thưởng (Loyalty Points) cho khách hàng
 * trong hệ thống TechShop.
 *
 * <p>Quy tắc nghiệp vụ:</p>
 * <ul>
 *   <li>Cứ 100.000 VNĐ → 1 điểm thưởng (tính phần nguyên).</li>
 *   <li>Mua hàng cuối tuần (Thứ 7, Chủ Nhật) → nhân đôi điểm.</li>
 *   <li>Khách VIP → cộng thêm 20% (sau tất cả ưu đãi khác, làm tròn lên).</li>
 * </ul>
 */
public class LoyaltyPointCalculator {

    /**
     * Tính toán tổng điểm thưởng dựa trên giá trị đơn hàng và các chính sách ưu đãi.
     *
     * @param orderAmount Giá trị đơn hàng (VNĐ). Giá trị <= 0 hoặc < 100.000 sẽ trả về 0 điểm.
     * @param isWeekend   {@code true} nếu giao dịch diễn ra vào cuối tuần (Thứ 7 hoặc Chủ Nhật).
     * @param isVip       {@code true} nếu khách hàng thuộc hạng VIP.
     * @return Số điểm thưởng đạt được (số nguyên >= 0).
     */
    public int calculatePoints(double orderAmount, boolean isWeekend, boolean isVip) {
        // Edge case: Đơn hàng âm, bằng 0, hoặc chưa đạt ngưỡng 100.000 VNĐ → 0 điểm
        if (orderAmount < 100000) {
            return 0;
        }

        // Bước 1: Tính điểm cơ sở — cứ 100.000 VNĐ = 1 điểm (lấy phần nguyên)
        int basePoints = (int) (orderAmount / 100000);

        // Bước 2: Nếu mua vào cuối tuần → nhân đôi số điểm
        int currentPoints = isWeekend ? basePoints * 2 : basePoints;

        // Bước 3: Nếu là VIP → cộng thêm 20% và làm tròn lên (Math.ceil)
        if (isVip) {
            currentPoints = (int) Math.ceil(currentPoints * 1.2);
        }

        return currentPoints;
    }
}
```

**Class kiểm thử — LoyaltyPointCalculatorTest.java:**

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import static org.junit.jupiter.api.Assertions.assertEquals;

/**
 * Unit Test cho LoyaltyPointCalculator.
 * Bao phủ 5 kịch bản: ngày thường, cuối tuần, VIP, cuối tuần + VIP, edge cases.
 */
public class LoyaltyPointCalculatorTest {

    private LoyaltyPointCalculator calculator;

    @BeforeEach
    public void setUp() {
        calculator = new LoyaltyPointCalculator();
    }

    @Test
    @DisplayName("Ngày thường, không VIP: 250.000 VNĐ → 2 điểm")
    public void testStandardPurchase_NoVip_NotWeekend() {
        // 250.000 / 100.000 = 2 điểm cơ sở
        // Không phải cuối tuần → giữ nguyên 2 điểm
        // Không phải VIP → giữ nguyên 2 điểm
        int points = calculator.calculatePoints(250000, false, false);
        assertEquals(2, points);
    }

    @Test
    @DisplayName("Cuối tuần, không VIP: 300.000 VNĐ → 6 điểm")
    public void testWeekendPurchase_NoVip() {
        // 300.000 / 100.000 = 3 điểm cơ sở
        // Cuối tuần → 3 × 2 = 6 điểm
        // Không phải VIP → giữ nguyên 6 điểm
        int points = calculator.calculatePoints(300000, true, false);
        assertEquals(6, points);
    }

    @Test
    @DisplayName("Ngày thường, VIP: 400.000 VNĐ → 5 điểm (làm tròn lên)")
    public void testVipPurchase_NotWeekend() {
        // 400.000 / 100.000 = 4 điểm cơ sở
        // Không phải cuối tuần → giữ nguyên 4 điểm
        // VIP → 4 × 1.2 = 4.8 → ceil = 5 điểm
        int points = calculator.calculatePoints(400000, false, true);
        assertEquals(5, points);
    }

    @Test
    @DisplayName("Cuối tuần + VIP: 500.000 VNĐ → 12 điểm")
    public void testVipAndWeekendPurchase() {
        // 500.000 / 100.000 = 5 điểm cơ sở
        // Cuối tuần → 5 × 2 = 10 điểm
        // VIP → 10 × 1.2 = 12.0 → ceil = 12 điểm
        int points = calculator.calculatePoints(500000, true, true);
        assertEquals(12, points);
    }

    @Test
    @DisplayName("Edge cases: Âm, 0, dưới ngưỡng 100.000 VNĐ → 0 điểm")
    public void testEdgeCases_ZeroNegativeOrBelowThreshold() {
        // Đơn hàng giá trị âm
        assertEquals(0, calculator.calculatePoints(-50000, false, false));
        // Đơn hàng bằng 0
        assertEquals(0, calculator.calculatePoints(0, false, false));
        // Đơn hàng chưa đạt ngưỡng 100.000 VNĐ
        assertEquals(0, calculator.calculatePoints(99999, true, true));
    }
}
```

### Kiểm chứng logic nghiệp vụ từ kết quả AI:

| Kịch bản | Input | Phép tính | Kết quả | Đúng NV? |
|-----------|-------|-----------|---------|----------|
| Ngày thường, không VIP | 250k | 250k/100k = 2 | 2 điểm | ✅ |
| Cuối tuần, không VIP | 300k | 300k/100k = 3 → ×2 = 6 | 6 điểm | ✅ |
| Ngày thường, VIP | 400k | 400k/100k = 4 → ×1.2 = 4.8 → ceil = 5 | 5 điểm | ✅ |
| Cuối tuần + VIP | 500k | 500k/100k = 5 → ×2 = 10 → ×1.2 = 12 | 12 điểm | ✅ |
| Đơn hàng âm | -50k | < 100k | 0 điểm | ✅ |
| Đơn hàng = 0 | 0 | < 100k | 0 điểm | ✅ |
| Dưới ngưỡng | 99.999 | < 100k | 0 điểm | ✅ |

**Kết luận:** Toàn bộ code do AI sinh ra đều hoạt động đúng theo 3 luật nghiệp vụ từ Business Analyst và xử lý được các edge case (giá trị âm, bằng 0, dưới ngưỡng).
