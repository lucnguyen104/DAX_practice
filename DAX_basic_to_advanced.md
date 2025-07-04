## Script để check các column nằm ở các bảng nào

```dax
DEFINE
    VAR A = 
        SELECTCOLUMNS(
            INFO.COLUMNS(),
            "TableID", [TableID],
            "ColumnName", [SourceColumn],
            "TableName",
            VAR CurrentTableID = [TableID]
            RETURN
                CONCATENATEX(
                    FILTER(
                        INFO.TABLES(),
                        [ID] = CurrentTableID
                    ),
                    [Name], ","
                )
        )

    VAR B = 
        FILTER(
            A,
            VAR CurrentCol = [ColumnName]
            RETURN
                COUNTROWS(
                    FILTER(
                        A,
                        [ColumnName] = CurrentCol
                    )
                ) > 1 &&
                [ColumnName] <> "" &&
                CONTAINSSTRING([ColumnName], "Key")
        )

EVALUATE B
ORDER BY [ColumnName] ASC

EVALUATE 
    DISTINCT(
        SELECTCOLUMNS(
            INFO.TABLES(),
            "TableName", [Name],
            "Category", [Description]
        )
    )
ORDER BY [Category] DESC
```

## Chapter 6: Các hàm tính toán trong DAX

```dax
Countrows = COUNTROWS(Sales)
DISTINCTCOUNT = DISTINCTCOUNT(Sales[CustomerKey])
```

## Chapter 9: Các hàm bỏ bộ lọc

### ALLSELECTED - Example 1
```dax
ALLSELECTED - Example 1 = 
DIVIDE(
    [Sales Amount],
    CALCULATE(
        [Sales Amount],
        ALLSELECTED(Customer[Continent])
    )
)
```

### ALLSELECTED - Example 1 - Advance
```dax
ALLSELECTED - Example 1 - Advance = 
DIVIDE(
    [Sales Amount],
    CALCULATE(
        [Sales Amount],
        ALLSELECTED()
    )
)
```

### ALLSELECTED - Example 2
```dax
VAR SalesContributionByCategory = 
    DIVIDE(
        [Sales Amount],
        CALCULATE(
            [Sales Amount],
            ALLSELECTED('Product'[Category])
        )
    )

VAR SalesContributionByCountry = 
    DIVIDE(
        [Sales Amount],
        CALCULATE(
            [Sales Amount],
            ALLSELECTED('Product'[Category]),
            ALLSELECTED(Customer[Country])
        )
    )

VAR SalesContributionByContinent = 
    DIVIDE(
        [Sales Amount],
        CALCULATE(
            [Sales Amount],
            ALLSELECTED()
        )
    )

RETURN
    SWITCH(
        TRUE(),
        ISFILTERED('Product'[Category]), SalesContributionByCategory,
        ISFILTERED(Customer[Country]), SalesContributionByCountry,
        ISFILTERED(Customer[Continent]), SalesContributionByContinent
    )
```



## 9.7. Đáp án bài tập

### 12.1 - Tỷ lệ % lợi nhuận so với doanh thu
```dax
DIVIDE([#Profit], [#Revenue])
```

### 12.2 - Tỷ lệ % doanh thu so với chi phí
```dax
DIVIDE([#Revenue], [#Cost])
```

### 12.3 - Trung bình mỗi khách hàng mua bao nhiêu SKUs
```dax
DIVIDE(
    [#Number Of Product],
    [#Number Of Customer]
)
```

### 12.3 - Fix 1 - Trung bình số sản phẩm mỗi customer
```dax
AVERAGEX(
    VALUES(Customer[CustomerKey]),
    [#Number Of Product]
)
```

### 12.3 - Fix 2 - Trung bình SKUs mỗi customer qua các năm
```dax
VAR Result =
    CALCULATE(
        AVERAGEX(
            SUMMARIZE(Sales, 'Date'[Year]),
            [Chapter 6,7,8,9 - 12.3 - Fix 1]
        ),
        ALL('Date'[Year])
    )

RETURN
    IF([#Revenue] <> 0, Result)
```

### 13.1 - Tính tỷ lệ doanh thu theo danh mục
```dax
VAR LstCategory = VALUES('Product'[Category])

VAR Result = 
    DIVIDE(
        CALCULATE(
            [#Revenue],
            ALL('Product'),
            'Product'[Category] IN LstCategory
        ),
        CALCULATE(
            [#Revenue],
            ALL('Product')
        )
    )

RETURN
    Result
```

### Chapter 6,7,8,9 - 14.1

**Cách 1:**
```dax
// COUNTROWS(
//     FILTER(
//         CALCULATETABLE(
//             VALUES(Store[StoreKey]),
//             Store[Status] = "" 
//         ),
//         [Sales Amount] > 700
//     )
// )
```

**Cách 2:**
```dax
CALCULATE(
    // [#Number Of Store],
    COUNTROWS(Store),
    FILTER(
        VALUES(Store[StoreKey]),
        [Sales Amount] > 700
    ),
    Store[Status] = "" 
)
```

---

### Chapter 6,7,8,9 - 14.2

```dax
DIVIDE(
    [Chapter 6,7,8,9 - 14.1],
    CALCULATE(
        [#Number Of Store],
        Store[Status] = ""
    )
)
```

---

### Chapter 6,7,8,9 - 15.1

```dax
VAR CurrentYear = MAX('Date'[Year])
RETURN
    CALCULATE(
        [#Revenue],
        'Date'[Year] <= CurrentYear
    )
```

---

### Chapter 6,7,8,9 - 15.1 - Fix

```dax
VAR MinYear = MIN('Date'[Year])
VAR MaxYear = MAX('Date'[Year])
VAR CurrentYearViz = MAX('DateViz'[Year])

VAR Result =
    IF(
        CurrentYearViz >= MinYear && CurrentYearViz <= MaxYear,
        CALCULATE(
            [#Revenue],
            'Date'[Year] <= CurrentYearViz
        )
    )

RETURN
    Result
```

---

### Chapter 6,7,8,9 - 15.2

```dax
VAR CurrentYear = MAX('Date'[Year])
RETURN
    CALCULATE(
        [#Profit],
        'Date'[Year] <= CurrentYear
    )
```

---

### Chapter 6,7,8,9 - 15.3

```dax
VAR CurrentYear = MAX('Date'[Year])
RETURN
    CALCULATE(
        [#Number Of Customer],
        'Date'[Year] <= CurrentYear
    )
```


### Chapter 6,7,8,9 - 17 - Cách 1 = 

```dax
VAR ListCustomer = 

   ADDCOLUMNS(

      SUMMARIZE(Sales, Customer\[CustomerKey]),

      "@NumberOfNonSalesMonth",

      VAR ListSalesMonth = 

           CALCULATETABLE(

               FILTER(

                   SUMMARIZE(Sales,'Date'\[Year Month Number]),

                   \[Sales Amount] > 0

               )

           )

       VAR AllMonth = 

           VALUES('Date'\[Year Month Number])

       VAR NonSalesMonth = 

           EXCEPT(AllMonth,ListSalesMonth)

       RETURN

           COUNTROWS(NonSalesMonth)

   )

RETURN

   // CONCATENATEX(

   //     DISTINCT(SELECTCOLUMNS(ListCustomer,"Month",\[@NumberOfNonSalesMonth])),

   //     \[Month],", "

   // )

   SUMX(ListCustomer,\[@NumberOfNonSalesMonth])

   

  
```




### Chapter 6,7,8,9 - 17 - Cách 2 = 

```dax
VAR ListCustomer = 

   ADDCOLUMNS(

       SUMMARIZE(Sales, Customer\[CustomerKey]),

       "@NumberOfNonSalesMonth",

  

       COUNTROWS(

           FILTER(

               VALUES('Date'\[Year Month Number]),

               ISBLANK(\[Sales Amount]) || \[Sales Amount] <= 0

           )

       )

   )

RETURN

   // CONCATENATEX(

   //     DISTINCT(SELECTCOLUMNS(ListCustomer,"Month",\[@NumberOfNonSalesMonth])),

   //     \[Month],", "

   // )

   SUMX(ListCustomer,\[@NumberOfNonSalesMonth])
```






Chapter 6,7,8,9 - 17.2 = 

VAR ListCustomer = 

   ADDCOLUMNS(

       SUMMARIZE(Sales, Customer\[CustomerKey]),

       "@NumberOfNonSalesMonth",

  

       COUNTROWS(

           FILTER(

               VALUES('Date'\[Year Month Number]),

               ISBLANK(\[Sales Amount]) || \[Sales Amount] <= 0

           )

       )





   )

RETURN

   DIVIDE(

       SUMX(ListCustomer,\[@NumberOfNonSalesMonth]),

       COUNTROWS(

           FILTER(

               ListCustomer,

               \[@NumberOfNonSalesMonth] > 0

           )

       )

   )



Chapter 10: Các hàm bảng trong DAX



-- Sales của category theo từng năm 



DEFINE 

	VAR A =

		SUMMARIZE(Sales,'Product'\[Category],'Date'\[Year])



	VAR B = 

		ADDCOLUMNS(

			A, 

			"@Sales",

			\[Sales Amount],

			"@Sales1",

			CALCULATE(SUMX(Sales,\[Net Price]\*\[Quantity]))

		)



EVALUATE B





-- Avg Sales Per Month By Year, Category

-- Trung mỗi tháng trong 1 năm mỗi category bán được bao nhiêu

-- Update đếm tổng số tháng có phát sinh doanh số của category đó trong năm

DEFINE 

	VAR A =

		SUMMARIZE(Sales,'Product'\[Category],'Date'\[Year])



	VAR B = 

		ADDCOLUMNS(

			A, 

			"@Sales",

			\[Sales Amount],

			"@AvgSalesPerMonth",

			AVERAGEX(VALUES('Date'\[Month]),\[Sales Amount]),

			"NumberOfMonth",

			CALCULATE(COUNTROWS(SUMMARIZE(Sales,'Date'\[Month])))

		)



EVALUATE B





-- Đếm số lượng sản phẩm không doanh thu tại các country theo từng năm

DEFINE 

	VAR A = 

		SUMMARIZE(Sales,'Store'\[Country],'Date'\[Year])

		

EVALUATE A





-- Đếm số lượng sản phẩm không doanh thu tại các country theo từng năm



DEFINE 

	VAR A = 

		SUMMARIZE(Sales,'Store'\[Country],'Date'\[Year])

		

	VAR B = 

		ADDCOLUMNS(

			A,

			"ProductNoSales",

			COUNTROWS(

				FILTER(

					VALUES('Product'\[ProductKey]),

					ISBLANK(\[Sales Amount])

				)

			),

			"ProductSales",

			CALCULATE(DISTINCTCOUNT(Sales\[ProductKey])),

			"Total Product",

			DISTINCTCOUNT(Sales\[ProductKey])

		)

		

EVALUATE B





-- Đếm số lượng khách hàng mới trong năm của category tại mỗi country, %Growth New Customer 

DEFINE 

	MEASURE 'All Meaasures'\[New Customer] = 

		VAR ListCustomer = 

			CALCULATETABLE(

				ADDCOLUMNS(

					VALUES('Customer'\[CustomerKey]),

					"@FirstYear",

					YEAR(CALCULATE(MIN('Sales'\[Order Date])))

				),

				ALL('Date')

			)

		RETURN

			COUNTROWS(FILTER(ListCustomer,\[@FirstYear] IN VALUES('Date'\[Year])))

		

	VAR A = 

		SUMMARIZE(Sales,'Store'\[Country], 'Product'\[Category],'Date'\[Year])

		

	VAR B = 

		ADDCOLUMNS(

			A,

			"New Customer",

			\[New Customer],

			"%Growth New Customer",

			VAR CurrYr = \[Year]

			VAR PrevNew = CALCULATE(\[New Customer], 'Date'\[Year] = CurrYr-1)

			RETURN

               FORMAT(DIVIDE(\[New Customer]-PrevNew,PrevNew),"#%")

		)



EVALUATE B





-- Tính Sales của những khách hàng mới đó

-- Trung bình mỗi khách hàng mới mua bao nhiêu

-- Lấy Top 3 khách hàng có Sales cao nhất

-- Lấy ra Top 5 sản phẩm được mua nhiều nhất bởi khách hàng mới

-- Tính tỷ lệ %Sales của Top 5 sản phẩm đó so với total những sản phẩm được mua bởi khách hàng mới







DEFINE 

	MEASURE 'All Meaasures'\[New Customer] = 

		VAR ListCustomer = 

			CALCULATETABLE(

				ADDCOLUMNS(

					VALUES('Customer'\[CustomerKey]),

					"@FirstYear",

					YEAR(CALCULATE(MIN('Sales'\[Order Date])))

				),

				ALL('Date')

			)

		RETURN

			COUNTROWS(FILTER(ListCustomer,\[@FirstYear] IN VALUES('Date'\[Year])))

		

	VAR A = 

		SUMMARIZE(Sales,'Store'\[Country], 'Product'\[Category],'Date'\[Year])

		

	VAR B = 

		ADDCOLUMNS(

			A,

			"Sales Amount",\[Sales Amount],

			"New Customer",

			\[New Customer],

			"%Growth New Customer",

			VAR CurrYr = \[Year]

			VAR PrevNew = CALCULATE(\[New Customer], 'Date'\[Year] = CurrYr-1)

			RETURN

               FORMAT(DIVIDE(\[New Customer]-PrevNew,PrevNew),"#%"),

			"Total Sales New Customer",

			CALCULATE(

				\[Sales Amount],

				FILTER(

					VALUES('Customer'\[CustomerKey]),

					\[New Customer]

				)

			),

			"Average Sales Per Customer",

			AVERAGEX(

				FILTER(

					VALUES(Customer\[CustomerKey]),

					\[New Customer]

				),

				\[Sales Amount]

			),

			"Top 3 Customer",

			CONCATENATEX(TOPN(3,VALUES(Customer\[CustomerKey]),\[Sales Amount],DESC),\[CustomerKey],", "),

			"Top 5 Product By New Customer",

			CONCATENATEX(

				CALCULATETABLE(

					TOPN(5,VALUES('Product'\[ProductKey]),\[Sales Amount],DESC),

					FILTER(

						VALUES(Customer\[CustomerKey]),

						\[New Customer] 

					)

				),

				\[ProductKey],

				", "

			),

			"%Sales Top 5 Product by New Customer",

			VAR \_Numerator = 

				SUMX(

					CALCULATETABLE(

						TOPN(5,VALUES('Product'\[ProductKey]),\[Sales Amount],DESC),

						FILTER(

							VALUES(Customer\[CustomerKey]),

							\[New Customer] 

						)

					),

					\[Sales Amount]

				)

			VAR \_Denominator = \[Sales Amount]

			RETURN

				FORMAT(DIVIDE(\_Numerator, \_Denominator),"#%")	

		)

EVALUATE B



-- Xác định những Category có doanh thu trung bình trong 3 tháng liên tục bao gồm tháng hiện tại lớn hơn doanh thu trung bình cả năm

-- Lưu ý đủ 3 tháng mới tính, và xác định 3 tháng liên tục trong năm

-- Đạt yêu cầu: ✅

-- Không đạt yêu cầu: 💥





DEFINE 

	VAR A = SUMMARIZE(Sales,'Product'\[Category], 'Date'\[Year],'Date'\[Month Number], \[Year Month Number])

	VAR B = 

		ADDCOLUMNS(

			A,

			"@Sales",\[Sales Amount]

		)

	VAR C = 

		ADDCOLUMNS(

			B,

			"AvgSalesYear",

			VAR CurrentCategory = \[Category]

			VAR CurrentYear = \[Year]

			RETURN

				AVERAGEX(

					FILTER(B,\[Year] = CurrentYear \&\& \[Category] = CurrentCategory),

					\[@Sales]

				),

			"AvgSales3M",

			VAR CurrentCategory = \[Category]

			VAR CurrentYear = \[Year]

			VAR CurrentYM = \[Year Month Number]

			VAR Table3M = 

					FILTER(

						B, 

						\[Category] = CurrentCategory \&\& 

						\[Year] = CurrentYear 

						\&\& (\[Year Month Number] >= CurrentYM - 2 \&\& \[Year Month Number] <= CurrentYM)

					)

			RETURN

				IF(COUNTROWS(Table3M) = 3, AVERAGEX(Table3M,\[@Sales]))

		)

	VAR D = 

		ADDCOLUMNS(

			C,

			"Greater than Avg Sales Year",

			SWITCH(

				TRUE(),

				NOT(ISBLANK(\[AvgSales3M])) \&\& \[AvgSales3M] >= \[AvgSalesYear], "✅",

				NOT(ISBLANK(\[AvgSales3M])) \&\& \[AvgSales3M] < \[AvgSalesYear], "💥"

			)

			

		)

EVALUATE D

ORDER BY \[Category] ASC, \[Year] ASC, \[Month Number] ASC



Chapter 11: Các hàm xây dựng mối quan hệ ảo



Ví dụ hàm USERELATIONSHIP 1 = 

VAR ListOrders = VALUES(Sales\[Order Number])

VAR ListDeliveryOrders = 

   CALCULATETABLE(

       VALUES(Sales\[Order Number]),

       USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

   )

VAR Result = 

   COUNTROWS(INTERSECT(ListOrders,ListDeliveryOrders))

RETURN

   Result

   

   

   

Ví dụ hàm USERELATIONSHIP 1 - Advance = 

-- Sum lại từng tháng cho ngữ cảnh Year

VAR ListYM = 

   ADDCOLUMNS(

       VALUES('Date'\[Year Month Number]),

       "@Orders",

       VAR ListOrders = CALCULATETABLE(VALUES(Sales\[Order Number]))

       VAR ListDeliveryOrders = 

           CALCULATETABLE(

               VALUES(Sales\[Order Number]),

               USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

           )

       VAR Result = 

           COUNTROWS(INTERSECT(ListOrders,ListDeliveryOrders))

       RETURN

           Result

   )

RETURN

   // Result

   SUMX(ListYM,\[@Orders])



Ví dụ hàm USERELATIONSHIP 2 = 

VAR ListOrders = VALUES(Sales\[Order Number])

VAR ListDeliveryOrders = 

   CALCULATETABLE(

       VALUES(Sales\[Order Number]),

       USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

   )

VAR Result = 

   INTERSECT(ListOrders,ListDeliveryOrders)

RETURN

   CALCULATE(

       \[Sales Amount],

       Result

   )

   

   

Ví dụ hàm USERELATIONSHIP 2 - Advance = 

-- Sum lại từng tháng cho ngữ cảnh Year

VAR ListYM = 

   ADDCOLUMNS(

       VALUES('Date'\[Year Month Number]),

       "@Orders",

       VAR ListOrders = CALCULATETABLE(VALUES(Sales\[Order Number]))

       VAR ListDeliveryOrders = 

           CALCULATETABLE(

               VALUES(Sales\[Order Number]),

               USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

           )

       VAR Result = 

           INTERSECT(ListOrders,ListDeliveryOrders)

       RETURN

           CALCULATE(

               \[Sales Amount],

               Result

           )

   )

RETURN

   // Result

   SUMX(ListYM,\[@Orders])



**Ví dụ hàm USERELATIONSHIP 3 =** 

**VAR CurrentYM = CALCULATE(MAX('Date'\[YM Number]))**

**VAR PreviousYM =**  

    **CALCULATE(**

        **MAX('Date'\[YM Number]),**

        **ALL('Date'),**

        **'Date'\[YM Number] < CurrentYM**

    **)**

**VAR ListCurrentDeliveryOrder =** 

    **CALCULATETABLE(**

        **VALUES(Sales\[Order Number]),**

        **USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])**

    **)**



**VAR PreviousMonthOrder =** 

    **CALCULATETABLE(**

        **VALUES(Sales\[Order Number]),**

        **ALL('Date'),**

        **'Date'\[YM Number] = PreviousYM**

    **)**



**VAR Result =** 

    **INTERSECT(ListCurrentDeliveryOrder,PreviousMonthOrder)**

**RETURN**

    **COUNTROWS(Result)**

 

 

 

 

 **Ví dụ hàm USERELATIONSHIP 3 - Advance =** 

**VAR ListYM =** 

    **ADDCOLUMNS(**

        **VALUES('Date'\[YM Number]),**

        **"@Orders",**

        **VAR CurrentYM = CALCULATE(MAX('Date'\[YM Number]))**

        **VAR PreviousYM =**  

            **CALCULATE(**

                **MAX('Date'\[YM Number]),**

                **ALL('Date'),**

                **'Date'\[YM Number] < CurrentYM**

            **)**

        **VAR ListCurrentDeliveryOrder =** 

            **CALCULATETABLE(**

                **VALUES(Sales\[Order Number]),**

                **USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])**

            **)**



        **VAR PreviousMonthOrder =** 

            **CALCULATETABLE(**

                **VALUES(Sales\[Order Number]),**

                **ALL('Date'),**

                **'Date'\[YM Number] = PreviousYM**

            **)**



        **VAR Result =** 

            **INTERSECT(ListCurrentDeliveryOrder,PreviousMonthOrder)**

        **RETURN**

            **COUNTROWS(Result)**

    **)**

**RETURN**

    **SUMX(ListYM,\[@Orders])**

 

**CHAPTER 14: CÁC HÀM SQL TRONG DAX**



**DEFINE**

	**VAR A = SUMMARIZE('Sales','Product'\[Category],'Date'\[Year])**

	**VAR B =** 

		**ADDCOLUMNS(**

			**A,**

			**"@Sales",**

			**\[Sales Amount]**

		**)**

		

	**-- Ví dụ về hàm OFFSET (Lấy Sales Previous Year)**

	**VAR C =**

		**ADDCOLUMNS(**

			**B,**

			**"@Sales PY",**

			**VAR SalesPY =** 

				**OFFSET(**

					**-1,**

					**B,**

					**ORDERBY(\[Year],ASC),**

					**PARTITIONBY(\[Category]),**

					**MATCHBY(\[Year])**

				**)**

			**RETURN**

				**SUMX(SalesPY,\[@Sales])**

		**)**

	

	**-- Ví dụ hàm Index (Lấy Sales của năm thứ 2 kinh doanh của mỗi Category)**

	

	**VAR D =** 

		**ADDCOLUMNS(**

			**C,**

			**"@Sales Second Year",**

			**VAR SalesSecondYear =** 

				**INDEX(2,C,ORDERBY(\[Year],ASC),PARTITIONBY(\[Category]),MATCHBY(\[Year]))**

			**RETURN**

				**SUMX(SalesSecondYear,\[@Sales])**

		**)**

	

	**-- Ví dụ hàm WINDOW (Tính Sales AVG 3Year)**

	**VAR E =** 

		**ADDCOLUMNS(**

			**D,**

			**"@AVG\_3Year",**

			**VAR Sales3Year =** 

				**WINDOW(**

					**-2,REL,**

					**0,REL,**

					**D,**

					**ORDERBY(\[Year],ASC),PARTITIONBY(\[Category]),MATCHBY(\[Year])**

				**)**

			**RETURN**

				**AVERAGEX(Sales3Year,\[@Sales])**

		**)**

**EVALUATE E**

**ORDER BY \[Category], \[Year]** 



**Sales AVG 3 Year (WINDOW) =** 

**VAR LastYear = MAX('Date'\[Year])**

**VAR List3Year =** 

    **WINDOW(**

        **-2,REL,**

        **0,REL,**

        **CALCULATETABLE(**

            **VALUES('Date'\[Year]),**

            **ALL('Date'),**

            **'Date'\[Year] <= LastYear**

        **),**

        **ORDERBY(\[Year])**

    **)**

**RETURN**

    **CALCULATE(**

        **AVERAGEX(List3Year,\[Sales Amount]),**

        **ALL('Date')**

    **)**



**Sales AVG 3 Year (TOPN) =** 

**VAR LastYear = MAX('Date'\[Year])**

**VAR List3Year =** 

    **TOPN(**

        **3,**

        **CALCULATETABLE(**

            **VALUES('Date'\[Year]),**

            **ALL('Date'),**

            **'Date'\[Year] <= LastYear**

        **),**

        **\[Year], DESC**

    **)**

**RETURN**

    **// CONCATENATEX(FILTER(List3Year,NOT(ISBLANK(\[Year]))) ,\[Year],", ")**

    **AVERAGEX(List3Year,\[Sales Amount])**



**Chapter 15: Đáp án tập Chương 10, 11**



**DEFINE**

	**VAR A = SUMMARIZE(Sales,'Product'\[Category],'Date'\[Year])**

	

	**VAR \_Cau1\_Phan1 =** 

		**ADDCOLUMNS(**

			**A,**

			**"@Sales",**

			**\[Sales Amount]**

		**)**

		

		

	**VAR \_Cau2\_Phan1 =** 

		**ADDCOLUMNS(**

			**\_Cau1\_Phan1,**

			**"CategoryPercentage",**

			**VAR \_CurrentYear = \[Year]**

			**VAR \_SalesYear =** 

				**SUMX(**

					**FILTER(**

						**\_Cau1\_Phan1,**

						**\[Year] = \_CurrentYear**

					**),**

					**\[@Sales]**

				**)**

			**VAR \_Result =** 

				**DIVIDE(\[@Sales],\_SalesYear)**

			**RETURN \_Result**

		**)**

		

		

	**VAR \_Cau3\_Phan1 =** 

		**ADDCOLUMNS(**

			**\_Cau2\_Phan1,**

			**"CategoryPercentageMeasure",**

			**DIVIDE(**

				**\[Sales Amount],**

				**CALCULATE(**

					**\[Sales Amount],**

					**ALL('Product'\[Category])**

				**)**

			**)**

			

		**)**

		

	**VAR \_Cau4\_Phan1 =** 

		**GROUPBY(\_Cau3\_Phan1,\[Category],"@Sales",SUMX(CURRENTGROUP(),\[@Sales]))**



**EVALUATE \_Cau4\_Phan1**



	

**DEFINE**	

	**VAR \_Cau7\_Phan1 =** 

		**ADDCOLUMNS(**

			**SUMMARIZE(Sales,Product\[Brand], Promotion\[Promotion Category], 'Date'\[Calendar Year Month]),**

			**"%DT\_Campaign",**

			**VAR \_CurrentYear= VALUE(RIGHT(\[Calendar Year Month],4))**

			**RETURN**

				**DIVIDE(**

					**\[Sales Amount],**

					**CALCULATE(**

						**\[Sales Amount],**

						**ALL('Date'),**

						**ALL('Product'\[Brand]),**

						**'Date'\[Calendar Year Number] = \_CurrentYear**

					**)**

				**)**

		**)**



	**VAR \_Cau8\_Phan1 =** 

		**GROUPBY(\_Cau7\_Phan1,\[Brand],\[Calendar Year Month],"Group\_Brand\_YM",SUMX(CURRENTGROUP(),\[@Sales]))**



	**VAR \_Cau9\_Phan1 =** 

		**ADDCOLUMNS(**

			**SUMMARIZE(Sales,Customer\[CountryRegion],'Date'\[Calendar Year Month Number]),**

			**"@Sales",\[Sales Amount],**

			**"AccumulatedSales1",**

			**VAR CurrentCYM = \[Calendar Year Month Number]**

			**RETURN**

				**CALCULATE(**

					**\[Sales Amount],**

					**ALL('Date'),**

					**'Date'\[Calendar Year Month Number] <= CurrentCYM**

				**),**

			**"AccumulatedSales2",**

				**SUMX(**

					**WINDOW(**

						**0,ABS,**

						**0,REL,**

						**ALL('Date'\[Calendar Year Month Number]),**

						**ORDERBY('Date'\[Calendar Year Month Number],ASC)**

					**),**

					**\[Sales Amount]**

				**)**

		**)**



		**VAR \_Cau9\_Phan1\_1 =** 

			**ADDCOLUMNS(**

				**\_Cau9\_Phan1,**

				**"@CheckGrow",**

			

				**VAR CurrentSales = \[Sales Amount]**

				**VAR PreviousSales =** 

					**SUMX(**

						**OFFSET(-1,\_Cau9\_Phan1,ORDERBY(\[Calendar Year Month Number],ASC), PARTITIONBY(\[CountryRegion])),**

						**\[Sales Amount]**

					**)**

				**RETURN**	

					**IF(CurrentSales > PreviousSales,1,0)**

			**)**

			

		**VAR \_Cau10\_Phan1\_Cach1 =** 

			**ADDCOLUMNS(**

				**\_Cau9\_Phan1\_1,**

				**"@A",**

				**VAR \_CurrentCountryRegion = \[CountryRegion]**

				**VAR \_CurrentYM = \[Calendar Year Month Number]**

				**VAR LastActiveYM =** 

					**MAXX(**

						**FILTER(**

							**\_Cau9\_Phan1\_1,**

							**\[CountryRegion] = \_CurrentCountryRegion \&\&**

							**\[Calendar Year Month Number] < \_CurrentYM \&\&** 

							**\[@CheckGrow] = 1**

						**),**

						**\[Calendar Year Month Number]**

					**)**

				**VAR LastUnActiveYM =** 

					**MAXX(**

						**FILTER(**

							**\_Cau9\_Phan1\_1,**

							**\[CountryRegion] = \_CurrentCountryRegion \&\&**

							**\[Calendar Year Month Number] < \_CurrentYM \&\&**

							**\[@CheckGrow] = 0**

						**),**

						**\[Calendar Year Month Number]**

					**)**

				**RETURN**

					**IF(**

						**\[@CheckGrow] = 1,**

						**SUMX(**

							**FILTER(**

								**\_Cau9\_Phan1\_1,**

								**\[CountryRegion] = \_CurrentCountryRegion \&\&**

								**\[@CheckGrow] = 1 \&\&**

								**\[Calendar Year Month Number] > LastUnActiveYM \&\&**

								**\[Calendar Year Month Number] <= \_CurrentYM**

							**),**

							**\[@Sales]**

						**)**

					**)**

			**)**



**EVALUATE \_Cau10\_Phan1\_Cach1**

**ORDER BY \[CountryRegion], \[Calendar Year Month Number]**









**VAR \_Cau10\_Phan1\_Cach2 =** 

			**ADDCOLUMNS(**

				**\_Cau9\_Phan1,**

				**"AccumulatedSalesWithCondition",**

				**VAR CurrentYM = \[Calendar Year Month Number]**

				**VAR CurrentSales = \[@Sales]**

				**VAR \_ListYM =** 

						**CALCULATETABLE(**

							**VALUES('Date'\[Calendar Year Month Number]),**

							**'Date'\[Calendar Year Month Number] < CurrentYM**

						**)**

					

				**VAR LastUnActiveMonth =** 

						**MAXX(**

							**FILTER(**

								**\_ListYM,**

								**VAR PreviousSales =** 

									**CALCULATE(**

										**\[Sales Amount],**

										**OFFSET(-1,\_ListYM,ORDERBY(\[Calendar Year Month Number],ASC))**

									**)**

								**RETURN**

									**PreviousSales > CurrentSales**

							**),**

							**\[Calendar Year Month Number]**

						**)**

				**RETURN**

					**SUMX(**

						**FILTER(**

							**\_ListYM,**

							**\[Calendar Year Month Number] > LastUnActiveMonth \&\&** 

							**\[Calendar Year Month Number] <= CurrentYM**

						**),**

						**\[Sales Amount]**

					**)**

			**)**

			



**EVALUATE \_Cau10\_Phan1\_Cach2**

**ORDER BY \[CountryRegion], \[Calendar Year Month Number]**



**Cau 2 - Phan 2 =** 

**COUNTROWS(**

    **FILTER(**

        **VALUES('Product'\[Product Name]),**

        **\[Sales Amount] > 10000**

    **)**

**)**



**Cau 3 - Phan 2 =** 

**VAR ListProduct =** 

    **FILTER(**

        **ADDCOLUMNS(**

            **VALUES('Product'\[Product Name]),**

            **"@Sales",\[Sales Amount]**

        **),**

        **\[@Sales] > 10000**

    **)**

**RETURN**

    **AVERAGEX(ListProduct,\[@Sales])**



**Cau 4 - Phan 2 =** 

    **IF(**

        **\[Sales Amount] > 0,**

        **CONCATENATEX(**

            **TOPN(**

                **1,**

                **VALUES('Product'\[Product Name]),**

                **\[Sales Amount],DESC**

            **),**

            **\[Product Name]**

        **)**

    **)**



**Cau 5 - Phan 2 =** 

     **IF(**

        **\[Sales Amount] > 0,**

        **CONCATENATEX(**

            **TOPN(**

                **1,**

                **FILTER(**

                    **ADDCOLUMNS(**

                        **VALUES('Product'\[Product Name]),**

                        **"@Sales",\[Sales Amount]**

                    **),**

                    **\[@Sales] > 0**

                **),**

                **\[@Sales],ASC**

            **),**

            **\[Product Name]**

        **)**

    **)**



**Cau 6 - Phan 2 =** 

**VAR TotalSales = \[Sales Amount]**

**VAR Top3Category =**

    **TOPN (**

        **\[Top N Dynamic Value],**

        **ADDCOLUMNS (**

            **SUMMARIZE ( Sales, 'Product'\[Category] ),**

            **"@Sales", \[Sales Amount]**

        **),**

        **\[@Sales], DESC**

    **)**

**VAR AddRankNumber =**

    **UNION (**

        **ADDCOLUMNS (**

            **Top3Category,**

            **"@SalesContribution", DIVIDE ( \[@Sales], TotalSales ),**

            **"@Rank",**

                **VAR CurrentSales = \[@Sales]**

                **RETURN**

                    **RANKX ( Top3Category, \[@Sales], CurrentSales, DESC )**

        **),**

        **VAR OtherSales =**

            **TotalSales - SUMX ( Top3Category, \[@Sales] )**

        **RETURN**

            **FILTER(**

                **{**

                    **( "Other", OtherSales, DIVIDE ( OtherSales, TotalSales ), \[Top N Dynamic Value] + 1 )**

                **},**

                **\[Value2] > 0**

            **)**

    **)**

**VAR Result =** 

    **CONCATENATEX (**

        **AddRankNumber,**

        **\[@Rank] \& ". " \& \[Category] \& ": "**

            **\& FORMAT ( \[@Sales], "#,##0" ) \& " ("**

            **\& FORMAT ( \[@SalesContribution], "Percent" ) \& ")",**

        **UNICHAR ( 10 ),**

        **\[@Rank], ASC**

    **)**

**RETURN**

    **Result**



**Cau 10 =** 

**VAR Top2Customer =** 

    **TOPN(**

        **2,**

        **VALUES(Sales\[CustomerKey]),**

        **\[Sales Amount],**

        **DESC**

    **)**

**VAR Top3Product =** 

    **CALCULATETABLE(**

        **TOPN(**

            **3,**

            **ADDCOLUMNS(**

                **SUMMARIZE(Sales, 'Product'\[Product Name]),**

                **"@Sales",**

                **\[Sales Amount]**

            **),**

            **\[@Sales],DESC**

        **),**

        **INTERSECT(**

            **VALUES(Sales\[CustomerKey]),**

            **Top2Customer**

        **)**

    **)**

**VAR AddRank =** 

    **ADDCOLUMNS(**

        **Top3Product,**

        **"@Rank",**

        **RANK( DENSE,Top3Product, ORDERBY(\[@Sales],DESC))**

    **)**

**VAR Result =** 

    **CONCATENATEX(**

        **AddRank,**

        **\[@Rank]\&". "\&\[Product Name]\&" : "\&FORMAT(\[@Sales],"#,##0"),**

        **UNICHAR(10),**

        **\[@Rank], ASC**

    **)**

**RETURN**

    **Result**



**Cau 1 =** 

**-- Added Question: Số Sales trong 2 năm của các sản phẩm bán trong năm 2007, 2009 >= 20000**

**VAR ListYearCondition = {2007,2009}**

**RETURN**

    **COUNTROWS(**

        **CALCULATETABLE(**

            **FILTER(**

                **VALUES('Product'\[ProductKey]),**

                **VAR SalesYear =** 

                    **COUNTROWS(**

                        **INTERSECT(**

                            **CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number])),**

                            **ListYearCondition**

                        **)**

                    **)** 

                **VAR SalesTwoYear =** 

                    **CALCULATE(**

                        **\[Sales Amount],**

                        **TREATAS(ListYearCondition,'Date'\[Calendar Year Number])**

                    **)**

                **RETURN**

                    **SalesYear = 2 \&\& SalesTwoYear >= 20000**

            **),**

            **ALL('Date')**

        **)**

    **)**



**Cau 2 - Phan 3 =** 

**VAR ListYearCondition = {2007,2009}**

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES('Product'\[ProductKey]),**

            **VAR SalesYear =** 

                **COUNTROWS(**

                    **INTERSECT(**

                        **CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number])),**

                        **ListYearCondition**

                    **)**

                **)** 

                

            **RETURN**

                **SalesYear = 2**

        **),**

        **ALL('Date')**

    **)**



**RETURN**

    **CALCULATE(**

        **\[Sales Amount],**

        **// TREATAS(ListProduct,'Product'\[ProductKey])**

        **ListProduct**

    **)**



**Cau 3 - Phan 3 =** 

**VAR ListYearCondition = {2007,2009}**

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES('Product'\[ProductKey]),**

            **VAR \_ListYear = CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number]))**

            

            **VAR FirstCondition =** 

                **COUNTROWS(FILTER(\_ListYear,\[Calendar Year Number] = 2007)) = 1**

                

            **VAR SecondCondition =** 

                 **COUNTROWS(FILTER(\_ListYear,\[Calendar Year Number] = 2009)) = 0**

            **RETURN**

                **AND(FirstCondition,SecondCondition)**

        **),**

        **ALL('Date')**

    **)**



**RETURN**

    **COUNTROWS(ListProduct)**



**Cau 4 - Phan 3 =** 

**VAR ListYearCondition = {2007,2009}**

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES('Product'\[ProductKey]),**

            **VAR \_ListYear = CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number]))**

            

            **VAR FirstCondition =** 

                **COUNTROWS(FILTER(\_ListYear,\[Calendar Year Number] = 2007)) = 1**

                

            **VAR SecondCondition =** 

                 **COUNTROWS(FILTER(\_ListYear,\[Calendar Year Number] = 2009)) = 0**

            **RETURN**

                **AND(FirstCondition,SecondCondition)**

        **),**

        **ALL('Date')**

    **)**



**RETURN**

    **SUMX(ListProduct,\[Sales Amount])**



**Cau 5 - Phan 3 =** 

**VAR ListYearCondition = {2007,2008,2009}**

**RETURN**

    **COUNTROWS(**

        **CALCULATETABLE(**

            **FILTER(**

                **VALUES('Sales'\[ProductKey]),**

                **COUNTROWS(**

                    **CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number]))**

                **) = 3**

            **),**

            **ALL('Date'),**

            **TREATAS(ListYearCondition,'Date'\[Calendar Year Number])**

        **)**

    **)**



**Cau 7.1 - Phan 3 =** 

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES(Sales\[CustomerKey]),**

            **COUNTROWS(CALCULATETABLE(SUMMARIZE(Sales,Promotion\[Promotion Category]))) = 2**

        **),**

        **ALL('Date'),**

        **'Date'\[Calendar Year Number] IN {2007,2008}**

    **)**

**RETURN**

    **COUNTROWS(ListProduct)**



**Cau 7.2 - Phan 3 =** 

**VAR ListCondition = {2007,2008}**

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES(Sales\[CustomerKey]),**

            **VAR \_ListYear =** 

                **FILTER(**

                    **CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number])),**

                    **CALCULATE(COUNTROWS(SUMMARIZE(Sales,Promotion\[Promotion Category]))) = 2**

                **)**

            **RETURN**

                **COUNTROWS(\_ListYear) = COUNTROWS(ListCondition)**

        **),**

        **ALL('Date'),**

        **'Date'\[Calendar Year Number] IN {2007,2008}**

    **)**

**RETURN**

    **COUNTROWS(ListProduct)**



**Cau 8.1 - Phan 3 =** 

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES(Sales\[CustomerKey]),**

            **COUNTROWS(CALCULATETABLE(SUMMARIZE(Sales,Promotion\[Promotion Category]))) = 2**

        **),**

        **ALL('Date'),**

        **'Date'\[Calendar Year Number] IN {2007,2008}**

    **)**

**RETURN**

    **SUMX(ListProduct, \[Sales Amount])**



**Cau 8.2 - Phan 3 =** 

**VAR ListCondition = {2007,2008}**

**VAR ListProduct =** 

    **CALCULATETABLE(**

        **FILTER(**

            **VALUES(Sales\[CustomerKey]),**

            **VAR \_ListYear =** 

                **FILTER(**

                    **CALCULATETABLE(SUMMARIZE(Sales,'Date'\[Calendar Year Number])),**

                    **CALCULATE(COUNTROWS(SUMMARIZE(Sales,Promotion\[Promotion Category]))) = 2**

                **)**

            **RETURN**

                **COUNTROWS(\_ListYear) = COUNTROWS(ListCondition)**

        **),**

        **ALL('Date'),**

        **'Date'\[Calendar Year Number] IN {2007,2008}**

    **)**

**RETURN**

    **SUMX(ListProduct, \[Sales Amount])**



**Chapter 11 - Cau 1 =** 

**VAR Top5Customer =** 

    **CALCULATETABLE(**

        **TOPN(5,VALUES(Sales\[CustomerKey]),\[Sales Amount],DESC),**

        **ALL('Date'),**

        **INDEX(**

            **1,**

            **CALCULATETABLE(VALUES('Date'\[Year]),ALL('Date'),'Date'\[IsTransaction] = TRUE()),**

            **ORDERBY(\[Year],ASC)**

        **)**

    **)**

**RETURN**

    **// CALCULATE(**

    **//     \[Sales Amount],**

    **//     TREATAS(Top5Customer,Sales\[CustomerKey])**

    **// )**

    **SUMX(Top5Customer,\[Sales Amount])**



**Chapter 11 - Cau 2 =** 

**VAR Top5Customer =** 

    **CALCULATETABLE(**

        **TOPN(5,VALUES(Sales\[CustomerKey]),\[Sales Amount],DESC),**

        **ALL('Date'),**

        **INDEX(**

            **1,**

            **CALCULATETABLE(VALUES('Date'\[Year]),ALL('Date'),'Date'\[IsTransaction] = TRUE()),**

            **ORDERBY(\[Year],ASC)**

        **)**

    **)**

**VAR ListBrand =** 

    **CALCULATETABLE(**

        **SUMMARIZE(Sales,'Product'\[Brand]),**

        **TREATAS(Top5Customer,Sales\[CustomerKey])**

    **)**

**RETURN**

    **SUMX(ListBrand,\[Sales Amount])**



**Chapter 11 - Cau 3 =** 

	**VAR Top3ProductByStore =** 

		**GENERATE(**

			**VALUES(Sales\[StoreKey]),**

			**VAR Top3Product =** 

				**TOPN(**

					**3,VALUES(Sales\[ProductKey]),\[Sales Amount],DESC**

				**)**

			

			**VAR \_Result =** 

				**CALCULATETABLE(**

					**ADDCOLUMNS(**

						**VALUES(Sales\[Currency Code]),**

						**"@Sales",**

						**CALCULATE(SUM(Sales\[Net Sales Exchange]))**

					**),**

					**Top3Product**

				**)**

				

			**RETURN**

				**\_Result**

		**)**

**VAR GroupData =** 

    **GROUPBY(Top3ProductByStore,\[Currency Code],"@Sales",SUMX(CURRENTGROUP(),\[@Sales]))**

**VAR Result =** 

    **CONCATENATEX(**

        **GroupData,**

        **\[Currency Code]\&": "\&**

        **FORMAT(\[@Sales],"#,##0"),**

        **UNICHAR(10),**

        **\[@Sales],DESC**

    **)**

**RETURN**

    **Result**



**Chapter 11 - Cau 5 =** 

    **SUMX(**

        **TOPN(3,VALUES(Sales\[CustomerKey]),\[Sales Amount],DESC),**

        **\[Sales Amount]**

    **)**



**Chapter 11 - Cau 6 =** 

**VAR FirstYear =** 

    **CALCULATE(**

        **MIN('Date'\[Year]),**

        **ALL('Date'),**

        **'Date'\[IsTransaction] = TRUE()**

    **)**



**VAR Top5Customer =** 

    **CALCULATETABLE(**

        **TOPN(5,VALUES(Sales\[CustomerKey]),\[Sales Amount],DESC),**

        **ALL('Date'),**

        **'Date'\[Year] = FirstYear**

    **)**



**VAR Top1Brand =** 

    **CALCULATETABLE(**

        **TOPN(1,VALUES('Product'\[Brand]),\[Sales Amount],DESC),**

        **ALL('Date'),**

        **'Date'\[Year] = FirstYear,**

        **TREATAS(Top5Customer,'Sales'\[CustomerKey])**

    **)**



**RETURN**

    **SUMX(Top1Brand,\[Sales Amount])**



**DEFINE**

	**VAR MaxYear =  CALCULATE(MAX('Date'\[Year]),'Date'\[IsTransaction] = TRUE())**

	**VAR ListYear =** 

		**ADDCOLUMNS(**

			**CALCULATETABLE(**

				**SUMMARIZECOLUMNS('Customer'\[CustomerKey],'Date'\[Year]),**

				**'Date'\[IsTransaction]= TRUE(),**

				**'Date'\[Year] <= MaxYear**

			**),**

			**"@Sales",**

			**\[Sales Amount]**

		**)**

	**VAR Result =** 

		**ADDCOLUMNS(**

			**ListYear,**

			**"@Index",**

			**VAR \_CurrentYear = \[Year]**

			**VAR \_CustomerKey = \[CustomerKey]**

			**VAR \_CurrentSales = \[@Sales]**

			**VAR \_LastestUnactiveMonth =** 

				**MAXX(**

					**FILTER(**

						**ListYear,**

						**\[Year] < \_CurrentYear \&\& \[CustomerKey] = \_CustomerKey \&\&**

						**ISBLANK(\[@Sales])** 

					**),**

					**\[Year]**

				**)**

			**VAR \_Result =** 

				**COUNTROWS(**

					**FILTER(**

						**ListYear,**

						**\[Year] <= \_CurrentYear \&\& \[Year] > \_LastestUnactiveMonth \&\& \[CustomerKey] = \_CustomerKey \&\&**

						**NOT(ISBLANK(\[@Sales]))**

					**)**

				**)**

			**RETURN**

				**\_Result**

		**)**

**EVALUATE Result**

**ORDER BY \[CustomerKey], \[Year]**

