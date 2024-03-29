# DBA_QueryPlan


# ICI 

Recursive Plan 透過 Intermediate Codes Interpreter 中間程式解譯器轉成『 標準中間格式 』，則即便查詢引擎和存取引擎語言不同，也能無縫地執行。

                                 存取介面 - 存取引擎

                                       \ | /

                                        ICI

                                       / | \

                                 執行計畫 - 執行引擎

                                      |  |  |

                                 查詢計畫 (預先建立)



# 查詢計畫的建置

計畫就是為達成一目的，而設立可重復性使用的計畫，所以 Plan 在此亦稱為 Recursive。

查詢最佳化作為查詢引擎中理論性最強的一部分，其效能不段加強一直是所有 DBA 希望達成的目標。隨著存儲技術的發展，其所支援的功能不斷增加，也同時影響查詢最佳化的設計。

設想一下，目前正好有一條查詢路徑，其中包含了表明該查詢路徑類型的類型和基表等等資訊，並且依據不同的查詢路徑類型，為其建立對應的查詢計畫。

範例：

          create_scan_plan

          create_seqscan_plan

          create_join_plan

          create_mergejoin_plan

          create_hashjoin_plan

# 查詢計畫的閱讀

當 DBA 執行一筆查詢敘述後，獲得了錯誤的查詢結果，在系統中增加了一種新的最佳化方法，並對應地增加了數個不同類型的查詢計畫。爾後，由 DBA 檢視其中耗費時間長的查詢效能，其中瓶頸出在何處（哪個步驟那個階段？）。

>>>

                           Query Plan
           
      ---------------------------------------------------------------
      
      Sort (cost=   rows=2    width=30)
      
            Sort Key: (avg(sc.scroe))
            
            -> HashAggregate  (cost=   rows=2    width=30)
            
                   Group Key: class.classno, class.classname
                   Filter: (avg(sc.score) > 60::numeric)
                   -> Nested Loop (cost=   rows=55    width=30)
                         Join Filter: (SubPlan 1)
                         -> Hash Join (cost=   rows=37    width=17)
                              Hash Cond: ((sc.cno)::text = (course.cno)::text)
                              //-> ...
                              
                         -> Materialize ()
                         -> Seq Scan on class
                         -> Seq Scan on student
                                   

# 查詢計畫的思考

在查詢計畫建立過程中，其依據的是一條最優查詢路徑，並且以 recursive 的方式，對該查詢存取路徑建立對應的查詢計畫。

整個建立過程步驟如下：

1. binary tree search.  

          QueryPlan
                SubQueryPlan1
                SubQueryPlan2


>>>

          一顆查詢存取 (路徑) 樹由相互獨立的兩顆子（查詢存取路徑）樹組成。

2.  check the leafs to root. 

          對該查詢存取樹，建置出查詢計畫樹。（藉由檢查葉子節點到根節點的方式）

3. threads. 

          倘若子查詢樹均採用執行緒方式，則由兩個執行緒獨立地完成對應子查詢『計畫』樹的建置。
# 執行計畫

如上述，透過查詢存取路徑來建置查詢計畫，方才透過執行引擎進而解釋查詢計畫，並且執行。


