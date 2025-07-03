Script để check các column nằm ở các bảng nào ?

DEFINE

&nbsp;	VAR A = 

&nbsp;		SELECTCOLUMNS(

&nbsp;			INFO.COLUMNS(),

&nbsp;			"TableID",\[TableID],

&nbsp;			"ColumnName",\[SourceColumn],

&nbsp;			"TableName",

&nbsp;			VAR CurrentTableID = \[TableID]

&nbsp;			RETURN

&nbsp;				CONCATENATEX(

&nbsp;					FILTER(

&nbsp;						INFO.TABLES(),

&nbsp;						\[ID] = CurrentTableID

&nbsp;					),

&nbsp;					\[Name],","

&nbsp;				)

&nbsp;		)

&nbsp;	VAR B = 

&nbsp;		FILTER(

&nbsp;			A,

&nbsp;			VAR CurrentCol = \[ColumnName]

&nbsp;			RETURN

&nbsp;				COUNTROWS(

&nbsp;					FILTER(

&nbsp;						A,

&nbsp;						\[ColumnName] = CurrentCol

&nbsp;					)

&nbsp;				) > 1 \&\& \[ColumnName] <> "" 

&nbsp;				\&\& CONTAINSSTRING(\[ColumnName],"Key")

&nbsp;		)		



EVALUATE B ORDER BY \[ColumnName] ASC



EVALUATE 

&nbsp;	DISTINCT(

&nbsp;		SELECTCOLUMNS(

&nbsp;			INFO.TABLES(),

&nbsp;			"TableName",\[Name],

&nbsp;			"Category",\[Description]

&nbsp;		)

&nbsp;	)

ORDER BY \[Category] DESC



Chapter 6: Các hàm tính toán trong DAX



Demo Countrows = COUNTROWS(Sales)

Demo DISTINCTCOUNT = DISTINCTCOUNT(Sales\[CustomerKey])



Chapter 9: Các hàm bỏ bộ lọc



ALLSELECTED - Example 1 = 

DIVIDE(

&nbsp;   \[Sales Amount],

&nbsp;   CALCULATE(

&nbsp;       \[Sales Amount],

&nbsp;       ALLSELECTED(Customer\[Continent])

&nbsp;   )

)



Đoạn Script nâng cao và dynamic theo Column/Field



ALLSELECTED - Example 1 - Advance = 

DIVIDE(

&nbsp;   \[Sales Amount],

&nbsp;   CALCULATE(

&nbsp;       \[Sales Amount],

&nbsp;       ALLSELECTED()

&nbsp;   )

)



ALLSELECTED - Example 2 = 

VAR SalesContributionByCategory = 

&nbsp;   DIVIDE(

&nbsp;       \[Sales Amount],

&nbsp;       CALCULATE(

&nbsp;           \[Sales Amount],

&nbsp;           ALLSELECTED('Product'\[Category])

&nbsp;       )

&nbsp;   )



VAR SalesContributionByCountry = 

&nbsp;   DIVIDE(

&nbsp;       \[Sales Amount],

&nbsp;       CALCULATE(

&nbsp;           \[Sales Amount],

&nbsp;           ALLSELECTED('Product'\[Category]),

&nbsp;           ALLSELECTED(Customer\[Country])

&nbsp;       )

&nbsp;   )

VAR SalesContributionByContinent = 

&nbsp;   DIVIDE(

&nbsp;       \[Sales Amount],

&nbsp;       CALCULATE(

&nbsp;           \[Sales Amount],

&nbsp;           ALLSELECTED()

&nbsp;       )

&nbsp;   )

RETURN

&nbsp;   SWITCH(

&nbsp;       TRUE(),

&nbsp;       ISFILTERED('Product'\[Category]),SalesContributionByCategory,

&nbsp;       ISFILTERED(Customer\[Country]),SalesContributionByCountry,

&nbsp;       ISFILTERED(Customer\[Continent]),SalesContributionByContinent

&nbsp;   )



9.7. Đáp án bài tập



Chapter 6,7,8,9 - 12.1 = 

-- Tỷ lệ % lợi nhuận so với doanh thu 

DIVIDE(\[#Profit],\[#Revenue])



Chapter 6,7,8,9 - 12.2 = 

-- Tỷ lệ % doanh thu so với chi phí 

DIVIDE(\[#Revenue],\[#Cost])



Chapter 6,7,8,9 - 12.3 = 

-- Trung bình mỗi khách hàng mua bao nhiêu SKUs 

DIVIDE(

&nbsp;   \[#Number Of Product],

&nbsp;   \[#Number Of Customer]

)



Chapter 6,7,8,9 - 12.3 - Fix 1 = 

-- Tính ra cho từng customer và trung bình cái số Customer đó lại

AVERAGEX(

&nbsp;   VALUES(Customer\[CustomerKey]),

&nbsp;   \[#Number Of Product]

)



Chapter 6,7,8,9 - 12.3 - Fix 2 = 

-- Tính trung bình một khách hàng mua bao nhiêu SKUs qua các năm

VAR Result =

&nbsp;   CALCULATE(

&nbsp;       -- Tính trung bình qua các năm

&nbsp;       AVERAGEX(

&nbsp;           -- Liệt kê ra các Year có Sales

&nbsp;           SUMMARIZE(Sales,'Date'\[Year]),

&nbsp;           \[Chapter 6,7,8,9 - 12.3 - Fix 1]

&nbsp;       ),

&nbsp;       -- Bỏ bộ lọc của Year

&nbsp;       ALL('Date'\[Year])

&nbsp;   )

RETURN

&nbsp;   -- Loại bỏ những trường hợp Year bị blank

&nbsp;   IF(\[#Revenue] <> 0,Result)

&nbsp;   

&nbsp;   

Chapter 6,7,8,9 - 13.1 = 

VAR LstCategory = VALUES('Product'\[Category])

VAR Result = 

&nbsp;   DIVIDE(

&nbsp;       CALCULATE(

&nbsp;           \[#Revenue],

&nbsp;           ALL('Product'),

&nbsp;           // VALUES('Product'\[Category])

&nbsp;           'Product'\[Category] IN LstCategory

&nbsp;       ),

&nbsp;       CALCULATE(

&nbsp;           \[#Revenue],

&nbsp;           ALL('Product')

&nbsp;       )

&nbsp;   )

RETURN

&nbsp;   // COUNTROWS(LstCategory)

&nbsp;   Result

&nbsp;   

&nbsp;   

&nbsp;Chapter 6,7,8,9 - 14.1 = 

// Cách 1:

// COUNTROWS(

//     FILTER(

//         CALCULATETABLE(

//             VALUES(Store\[StoreKey]),

//             Store\[Status] = "" 

//         ),

//         \[Sales Amount] > 700

//     )

// )



// Cách 2:

CALCULATE(

&nbsp;   // \[#Number Of Store],

&nbsp;   COUNTROWS(Store),

&nbsp;   FILTER(

&nbsp;       VALUES(Store\[StoreKey]),

&nbsp;       \[Sales Amount] > 700

&nbsp;   ),

&nbsp;   Store\[Status] = "" 

)





Chapter 6,7,8,9 - 14.2 = 

DIVIDE(

&nbsp;   \[Chapter 6,7,8,9 - 14.1],

&nbsp;   CALCULATE(

&nbsp;       \[#Number Of Store],

&nbsp;       Store\[Status] = ""

&nbsp;   )

)





Chapter 6,7,8,9 - 15.1 = 

VAR CurrentYear = MAX('Date'\[Year])

RETURN

&nbsp;   CALCULATE(

&nbsp;       \[#Revenue],

&nbsp;       'Date'\[Year] <= CurrentYear

&nbsp;   )





Chapter 6,7,8,9 - 15.1 - Fix = 

VAR MinYear = MIN('Date'\[Year])

VAR MaxYear = MAX('Date'\[Year])

VAR CurrentYearViz = MAX('DateViz'\[Year])

VAR Result =

&nbsp;   IF(

&nbsp;       CurrentYearViz >= MinYear \&\& CurrentYearViz <= MaxYear,

&nbsp;       CALCULATE(

&nbsp;           \[#Revenue],

&nbsp;           'Date'\[Year] <= CurrentYearViz

&nbsp;       )

&nbsp;   )

RETURN

&nbsp;   Result

&nbsp;  

Chapter 6,7,8,9 - 15.2 = 

VAR CurrentYear = MAX('Date'\[Year])

RETURN

&nbsp;   CALCULATE(

&nbsp;       \[#Profit],

&nbsp;       'Date'\[Year] <= CurrentYear

&nbsp;   )



Chapter 6,7,8,9 - 15.3 = 

VAR CurrentYear = MAX('Date'\[Year])

RETURN

&nbsp;   CALCULATE(

&nbsp;       \[#Number Of Customer],

&nbsp;       'Date'\[Year] <= CurrentYear

&nbsp;   )









Chapter 6,7,8,9 - 17 - Cách 1 = 

VAR ListCustomer = 

&nbsp;   ADDCOLUMNS(

&nbsp;       SUMMARIZE(Sales, Customer\[CustomerKey]),

&nbsp;       "@NumberOfNonSalesMonth",

&nbsp;       VAR ListSalesMonth = 

&nbsp;           CALCULATETABLE(

&nbsp;               FILTER(

&nbsp;                   SUMMARIZE(Sales,'Date'\[Year Month Number]),

&nbsp;                   \[Sales Amount] > 0

&nbsp;               )

&nbsp;           )

&nbsp;       VAR AllMonth = 

&nbsp;           VALUES('Date'\[Year Month Number])

&nbsp;       VAR NonSalesMonth = 

&nbsp;           EXCEPT(AllMonth,ListSalesMonth)

&nbsp;       RETURN

&nbsp;           COUNTROWS(NonSalesMonth)

&nbsp;   )

RETURN

&nbsp;   // CONCATENATEX(

&nbsp;   //     DISTINCT(SELECTCOLUMNS(ListCustomer,"Month",\[@NumberOfNonSalesMonth])),

&nbsp;   //     \[Month],", "

&nbsp;   // )

&nbsp;   SUMX(ListCustomer,\[@NumberOfNonSalesMonth])

&nbsp;   

&nbsp;  





Chapter 6,7,8,9 - 17 - Cách 2 = 

VAR ListCustomer = 

&nbsp;   ADDCOLUMNS(

&nbsp;       SUMMARIZE(Sales, Customer\[CustomerKey]),

&nbsp;       "@NumberOfNonSalesMonth",

&nbsp;  

&nbsp;       COUNTROWS(

&nbsp;           FILTER(

&nbsp;               VALUES('Date'\[Year Month Number]),

&nbsp;               ISBLANK(\[Sales Amount]) || \[Sales Amount] <= 0

&nbsp;           )

&nbsp;       )



&nbsp;

&nbsp;   )

RETURN

&nbsp;   // CONCATENATEX(

&nbsp;   //     DISTINCT(SELECTCOLUMNS(ListCustomer,"Month",\[@NumberOfNonSalesMonth])),

&nbsp;   //     \[Month],", "

&nbsp;   // )

&nbsp;   SUMX(ListCustomer,\[@NumberOfNonSalesMonth])







Chapter 6,7,8,9 - 17.2 = 

VAR ListCustomer = 

&nbsp;   ADDCOLUMNS(

&nbsp;       SUMMARIZE(Sales, Customer\[CustomerKey]),

&nbsp;       "@NumberOfNonSalesMonth",

&nbsp;  

&nbsp;       COUNTROWS(

&nbsp;           FILTER(

&nbsp;               VALUES('Date'\[Year Month Number]),

&nbsp;               ISBLANK(\[Sales Amount]) || \[Sales Amount] <= 0

&nbsp;           )

&nbsp;       )



&nbsp;

&nbsp;   )

RETURN

&nbsp;   DIVIDE(

&nbsp;       SUMX(ListCustomer,\[@NumberOfNonSalesMonth]),

&nbsp;       COUNTROWS(

&nbsp;           FILTER(

&nbsp;               ListCustomer,

&nbsp;               \[@NumberOfNonSalesMonth] > 0

&nbsp;           )

&nbsp;       )

&nbsp;   )



Chapter 10: Các hàm bảng trong DAX



-- Sales của category theo từng năm 



DEFINE 

&nbsp;	VAR A =

&nbsp;		SUMMARIZE(Sales,'Product'\[Category],'Date'\[Year])



&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A, 

&nbsp;			"@Sales",

&nbsp;			\[Sales Amount],

&nbsp;			"@Sales1",

&nbsp;			CALCULATE(SUMX(Sales,\[Net Price]\*\[Quantity]))

&nbsp;		)



EVALUATE B





-- Avg Sales Per Month By Year, Category

-- Trung mỗi tháng trong 1 năm mỗi category bán được bao nhiêu

-- Update đếm tổng số tháng có phát sinh doanh số của category đó trong năm

DEFINE 

&nbsp;	VAR A =

&nbsp;		SUMMARIZE(Sales,'Product'\[Category],'Date'\[Year])



&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A, 

&nbsp;			"@Sales",

&nbsp;			\[Sales Amount],

&nbsp;			"@AvgSalesPerMonth",

&nbsp;			AVERAGEX(VALUES('Date'\[Month]),\[Sales Amount]),

&nbsp;			"NumberOfMonth",

&nbsp;			CALCULATE(COUNTROWS(SUMMARIZE(Sales,'Date'\[Month])))

&nbsp;		)



EVALUATE B





-- Đếm số lượng sản phẩm không doanh thu tại các country theo từng năm

DEFINE 

&nbsp;	VAR A = 

&nbsp;		SUMMARIZE(Sales,'Store'\[Country],'Date'\[Year])

&nbsp;		

EVALUATE A





-- Đếm số lượng sản phẩm không doanh thu tại các country theo từng năm



DEFINE 

&nbsp;	VAR A = 

&nbsp;		SUMMARIZE(Sales,'Store'\[Country],'Date'\[Year])

&nbsp;		

&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A,

&nbsp;			"ProductNoSales",

&nbsp;			COUNTROWS(

&nbsp;				FILTER(

&nbsp;					VALUES('Product'\[ProductKey]),

&nbsp;					ISBLANK(\[Sales Amount])

&nbsp;				)

&nbsp;			),

&nbsp;			"ProductSales",

&nbsp;			CALCULATE(DISTINCTCOUNT(Sales\[ProductKey])),

&nbsp;			"Total Product",

&nbsp;			DISTINCTCOUNT(Sales\[ProductKey])

&nbsp;		)

&nbsp;		

EVALUATE B





-- Đếm số lượng khách hàng mới trong năm của category tại mỗi country, %Growth New Customer 

DEFINE 

&nbsp;	MEASURE 'All Meaasures'\[New Customer] = 

&nbsp;		VAR ListCustomer = 

&nbsp;			CALCULATETABLE(

&nbsp;				ADDCOLUMNS(

&nbsp;					VALUES('Customer'\[CustomerKey]),

&nbsp;					"@FirstYear",

&nbsp;					YEAR(CALCULATE(MIN('Sales'\[Order Date])))

&nbsp;				),

&nbsp;				ALL('Date')

&nbsp;			)

&nbsp;		RETURN

&nbsp;			COUNTROWS(FILTER(ListCustomer,\[@FirstYear] IN VALUES('Date'\[Year])))

&nbsp;		

&nbsp;	VAR A = 

&nbsp;		SUMMARIZE(Sales,'Store'\[Country], 'Product'\[Category],'Date'\[Year])

&nbsp;		

&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A,

&nbsp;			"New Customer",

&nbsp;			\[New Customer],

&nbsp;			"%Growth New Customer",

&nbsp;			VAR CurrYr = \[Year]

&nbsp;			VAR PrevNew = CALCULATE(\[New Customer], 'Date'\[Year] = CurrYr-1)

&nbsp;			RETURN

&nbsp;               FORMAT(DIVIDE(\[New Customer]-PrevNew,PrevNew),"#%")

&nbsp;		)



EVALUATE B





-- Tính Sales của những khách hàng mới đó

-- Trung bình mỗi khách hàng mới mua bao nhiêu

-- Lấy Top 3 khách hàng có Sales cao nhất

-- Lấy ra Top 5 sản phẩm được mua nhiều nhất bởi khách hàng mới

-- Tính tỷ lệ %Sales của Top 5 sản phẩm đó so với total những sản phẩm được mua bởi khách hàng mới







DEFINE 

&nbsp;	MEASURE 'All Meaasures'\[New Customer] = 

&nbsp;		VAR ListCustomer = 

&nbsp;			CALCULATETABLE(

&nbsp;				ADDCOLUMNS(

&nbsp;					VALUES('Customer'\[CustomerKey]),

&nbsp;					"@FirstYear",

&nbsp;					YEAR(CALCULATE(MIN('Sales'\[Order Date])))

&nbsp;				),

&nbsp;				ALL('Date')

&nbsp;			)

&nbsp;		RETURN

&nbsp;			COUNTROWS(FILTER(ListCustomer,\[@FirstYear] IN VALUES('Date'\[Year])))

&nbsp;		

&nbsp;	VAR A = 

&nbsp;		SUMMARIZE(Sales,'Store'\[Country], 'Product'\[Category],'Date'\[Year])

&nbsp;		

&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A,

&nbsp;			"Sales Amount",\[Sales Amount],

&nbsp;			"New Customer",

&nbsp;			\[New Customer],

&nbsp;			"%Growth New Customer",

&nbsp;			VAR CurrYr = \[Year]

&nbsp;			VAR PrevNew = CALCULATE(\[New Customer], 'Date'\[Year] = CurrYr-1)

&nbsp;			RETURN

&nbsp;               FORMAT(DIVIDE(\[New Customer]-PrevNew,PrevNew),"#%"),

&nbsp;			"Total Sales New Customer",

&nbsp;			CALCULATE(

&nbsp;				\[Sales Amount],

&nbsp;				FILTER(

&nbsp;					VALUES('Customer'\[CustomerKey]),

&nbsp;					\[New Customer]

&nbsp;				)

&nbsp;			),

&nbsp;			"Average Sales Per Customer",

&nbsp;			AVERAGEX(

&nbsp;				FILTER(

&nbsp;					VALUES(Customer\[CustomerKey]),

&nbsp;					\[New Customer]

&nbsp;				),

&nbsp;				\[Sales Amount]

&nbsp;			),

&nbsp;			"Top 3 Customer",

&nbsp;			CONCATENATEX(TOPN(3,VALUES(Customer\[CustomerKey]),\[Sales Amount],DESC),\[CustomerKey],", "),

&nbsp;			"Top 5 Product By New Customer",

&nbsp;			CONCATENATEX(

&nbsp;				CALCULATETABLE(

&nbsp;					TOPN(5,VALUES('Product'\[ProductKey]),\[Sales Amount],DESC),

&nbsp;					FILTER(

&nbsp;						VALUES(Customer\[CustomerKey]),

&nbsp;						\[New Customer] 

&nbsp;					)

&nbsp;				),

&nbsp;				\[ProductKey],

&nbsp;				", "

&nbsp;			),

&nbsp;			"%Sales Top 5 Product by New Customer",

&nbsp;			VAR \_Numerator = 

&nbsp;				SUMX(

&nbsp;					CALCULATETABLE(

&nbsp;						TOPN(5,VALUES('Product'\[ProductKey]),\[Sales Amount],DESC),

&nbsp;						FILTER(

&nbsp;							VALUES(Customer\[CustomerKey]),

&nbsp;							\[New Customer] 

&nbsp;						)

&nbsp;					),

&nbsp;					\[Sales Amount]

&nbsp;				)

&nbsp;			VAR \_Denominator = \[Sales Amount]

&nbsp;			RETURN

&nbsp;				FORMAT(DIVIDE(\_Numerator, \_Denominator),"#%")	

&nbsp;		)

EVALUATE B



-- Xác định những Category có doanh thu trung bình trong 3 tháng liên tục bao gồm tháng hiện tại lớn hơn doanh thu trung bình cả năm

-- Lưu ý đủ 3 tháng mới tính, và xác định 3 tháng liên tục trong năm

-- Đạt yêu cầu: ✅

-- Không đạt yêu cầu: 💥





DEFINE 

&nbsp;	VAR A = SUMMARIZE(Sales,'Product'\[Category], 'Date'\[Year],'Date'\[Month Number], \[Year Month Number])

&nbsp;	VAR B = 

&nbsp;		ADDCOLUMNS(

&nbsp;			A,

&nbsp;			"@Sales",\[Sales Amount]

&nbsp;		)

&nbsp;	VAR C = 

&nbsp;		ADDCOLUMNS(

&nbsp;			B,

&nbsp;			"AvgSalesYear",

&nbsp;			VAR CurrentCategory = \[Category]

&nbsp;			VAR CurrentYear = \[Year]

&nbsp;			RETURN

&nbsp;				AVERAGEX(

&nbsp;					FILTER(B,\[Year] = CurrentYear \&\& \[Category] = CurrentCategory),

&nbsp;					\[@Sales]

&nbsp;				),

&nbsp;			"AvgSales3M",

&nbsp;			VAR CurrentCategory = \[Category]

&nbsp;			VAR CurrentYear = \[Year]

&nbsp;			VAR CurrentYM = \[Year Month Number]

&nbsp;			VAR Table3M = 

&nbsp;					FILTER(

&nbsp;						B, 

&nbsp;						\[Category] = CurrentCategory \&\& 

&nbsp;						\[Year] = CurrentYear 

&nbsp;						\&\& (\[Year Month Number] >= CurrentYM - 2 \&\& \[Year Month Number] <= CurrentYM)

&nbsp;					)

&nbsp;			RETURN

&nbsp;				IF(COUNTROWS(Table3M) = 3, AVERAGEX(Table3M,\[@Sales]))

&nbsp;		)

&nbsp;	VAR D = 

&nbsp;		ADDCOLUMNS(

&nbsp;			C,

&nbsp;			"Greater than Avg Sales Year",

&nbsp;			SWITCH(

&nbsp;				TRUE(),

&nbsp;				NOT(ISBLANK(\[AvgSales3M])) \&\& \[AvgSales3M] >= \[AvgSalesYear], "✅",

&nbsp;				NOT(ISBLANK(\[AvgSales3M])) \&\& \[AvgSales3M] < \[AvgSalesYear], "💥"

&nbsp;			)

&nbsp;			

&nbsp;		)

EVALUATE D

ORDER BY \[Category] ASC, \[Year] ASC, \[Month Number] ASC



Chapter 11: Các hàm xây dựng mối quan hệ ảo



Ví dụ hàm USERELATIONSHIP 1 = 

VAR ListOrders = VALUES(Sales\[Order Number])

VAR ListDeliveryOrders = 

&nbsp;   CALCULATETABLE(

&nbsp;       VALUES(Sales\[Order Number]),

&nbsp;       USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

&nbsp;   )

VAR Result = 

&nbsp;   COUNTROWS(INTERSECT(ListOrders,ListDeliveryOrders))

RETURN

&nbsp;   Result

&nbsp;   

&nbsp;   

&nbsp;   

Ví dụ hàm USERELATIONSHIP 1 - Advance = 

-- Sum lại từng tháng cho ngữ cảnh Year

VAR ListYM = 

&nbsp;   ADDCOLUMNS(

&nbsp;       VALUES('Date'\[Year Month Number]),

&nbsp;       "@Orders",

&nbsp;       VAR ListOrders = CALCULATETABLE(VALUES(Sales\[Order Number]))

&nbsp;       VAR ListDeliveryOrders = 

&nbsp;           CALCULATETABLE(

&nbsp;               VALUES(Sales\[Order Number]),

&nbsp;               USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

&nbsp;           )

&nbsp;       VAR Result = 

&nbsp;           COUNTROWS(INTERSECT(ListOrders,ListDeliveryOrders))

&nbsp;       RETURN

&nbsp;           Result

&nbsp;   )

RETURN

&nbsp;   // Result

&nbsp;   SUMX(ListYM,\[@Orders])



Ví dụ hàm USERELATIONSHIP 2 = 

VAR ListOrders = VALUES(Sales\[Order Number])

VAR ListDeliveryOrders = 

&nbsp;   CALCULATETABLE(

&nbsp;       VALUES(Sales\[Order Number]),

&nbsp;       USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

&nbsp;   )

VAR Result = 

&nbsp;   INTERSECT(ListOrders,ListDeliveryOrders)

RETURN

&nbsp;   CALCULATE(

&nbsp;       \[Sales Amount],

&nbsp;       Result

&nbsp;   )

&nbsp;   

&nbsp;   

Ví dụ hàm USERELATIONSHIP 2 - Advance = 

-- Sum lại từng tháng cho ngữ cảnh Year

VAR ListYM = 

&nbsp;   ADDCOLUMNS(

&nbsp;       VALUES('Date'\[Year Month Number]),

&nbsp;       "@Orders",

&nbsp;       VAR ListOrders = CALCULATETABLE(VALUES(Sales\[Order Number]))

&nbsp;       VAR ListDeliveryOrders = 

&nbsp;           CALCULATETABLE(

&nbsp;               VALUES(Sales\[Order Number]),

&nbsp;               USERELATIONSHIP('Date'\[Date],Sales\[Delivery Date])

&nbsp;           )

&nbsp;       VAR Result = 

&nbsp;           INTERSECT(ListOrders,ListDeliveryOrders)

&nbsp;       RETURN

&nbsp;           CALCULATE(

&nbsp;               \[Sales Amount],

&nbsp;               Result

&nbsp;           )

&nbsp;   )

RETURN

&nbsp;   // Result

&nbsp;   SUMX(ListYM,\[@Orders])



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

