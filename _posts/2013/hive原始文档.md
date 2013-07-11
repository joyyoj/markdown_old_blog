\chapter{HIVE}



\section{~HIVE~编译器~}

\begin{lstlisting}[language=SQL]
select * from src where key=1;
select count(*), count(1) from src limit 1;
select distinct key from src;
select key, count(*) from src group by key;
select key, count(distinct value) from src group by key having key>2;
select * from (select key, value from src1 union all select key, value from src2) tmp;
select src.key, src.value from src join src_tmp on src.key = src_tmp.key where src.value=``bcd'';

包含~subquery~的查询；
包含~view~的查询；

join~的类型：

\end{lstlisting}

\begin{lstlisting}[language=JAVA,frame=single]
class NodeProcessor {
  Object process(Node nd, Stack<Node> stack, NodeProcessorCtx procCtx, Object... nodeOutputs) throws SemanticException;
}
\end{lstlisting}

doPhase1(); //ASTTree $\rightarrow$ QB

getMetaData(); // 

genPlan(); // 生成~Operator~树；

optimize();

genMapRedTask();

\subsection{genPlan}
\begin{lstlisting}[language=JAVA]
Operator genPlan(QBExpr qbexpr) throws SemanticException {
if (qbexpr.getOpcode() == QBExpr.Opcode.NULLOP) {
return genPlan(qbexpr.getQB());
}
if (qbexpr.getOpcode() == QBExpr.Opcode.UNION) {
// 后序遍历递归生成
Operator qbexpr1Ops = genPlan(qbexpr.getQBExpr1());
Operator qbexpr2Ops = genPlan(qbexpr.getQBExpr2());
return genUnionPlan(qbexpr.getAlias(), qbexpr.getQBExpr1().getAlias(),
qbexpr1Ops, qbexpr.getQBExpr2().getAlias(), qbexpr2Ops);
}
}

Operator genPlan(QB qb) {
\begin{enumerate}
  \item 递归遍历子查询，生成子句的~plan，映射子句与对应的~OP；
  \item 遍历~Table~，生成~TableScan~算子，映射表与对应的~OP；
  \item genLateralViewPlans
  \item 处理~JOIN~算子
    \begin{itemize}
      \item 如果是~UNIQE JOIN，？
      \item mergeJoinTree
    \end{itemize}
  \item genBodyPlan
\end{enumerate}
}
处理~JOIN~算子 {
 // if any filters are present in the join tree, push them on top of the
 // table
 pushJoinFilters(qb, qb.getQbJoinTree(), aliasToOpInfo);
 genJoinPlan
}
TableRef~和子查询~处理上的区别有哪些？
a join b 时， left 是a, b right


// 这里要注意的是

genBodyPlan {
  
}

\end{lstlisting}

\subsection{预处理}
doPhase1
   1. Gets all the aliases for all the tables / subqueries and makes the appropriate mapping in aliasToTabs, aliasToSubq 
   2. Gets the location of the destination and names the clase ``inclause'' + i 
   3. Creates a map from a string representation of an aggregation tree to the actual aggregation AST
   4. Creates a mapping from the clause name to the select expression AST in destToSelExpr 
   5. Creates a mapping from a table alias to the lateral view AST's in aliasToLateralViews

\section{逻辑计划生成}

\begin{lstlisting}[language=JAVA]
public interface Transform {
  ParseContext transform(ParseContext pctx) throws SemanticException;
}
\end{lstlisting}

引申一个问题，一个~Task~的执行树，如何做到~MapOperator~setChildren()~设置正确的~TableScanOperator；
\subsection{GlobalLimitOptimizer}

\subsection{JoinReorder}

Join~次序的优化。
如果表的Tag~越大，则越后面处理。
这个优化目前必须用户指定关键字~STREAMTABLE~来指定~TAG~来能生效。
例如：
%

\begin{verbatim}
SELECT /*+ STREAMTABLE(a) */ a.key, a.val, c.key;
FROM T1 a JOIN src c ON c.key+1=a.key; 
\end{verbatim}

\subsection{ColumnPruner}
列裁减优化，如~RCFile~
\begin{itemize}
\item FIL\% 所需字段: 过滤条件需要的字段，去重~(~下同~)~。注意保留~column~在~RowSchema~的次序~(~每个~operatorgetSchema~)~
\item GBY\% 出现在~GroupByDesc~keys~的字段和聚合算子参数中引用的字段。
\item RS\%  a) 孩子节点是JoinOperator; b) 孩子节点不是JoinOperator，为出现在key中的字段+出现在value中的字段 
4）SelectOperator（ColumnPrunerSelectProc）所需字段为：4.1）如果有孩子节点为FileSinkOperator或者ScriptOperator或者UDTFOperator或者LimitOperator或者UnionOperator，那么从SelectOperator中获取所需字段。  4.2） 
5）JoinOperator（ColumnPrunerJoinProc）所需字段为：如果有孩子节点是FileSinkOperator，那么不处理。其他情况： 
6）MapJoinOperator（ColumnPrunerMapJoinProc） 
7）TableScanOperator（ColumnPrunerTableScanProc）所需字段为：孩子节点需要的字段。 
8）LateralViewJoinOperator（ColumnPrunerLateralViewJoinProc）
\end{itemize}

\subsection{简单优化}

设置了有几种模式，Normal~的优化是
~select columnList from table where partitionColumnFilter;
column~允许是虚拟列。(INPUT\_\_FILE\_\_NAME等)
Maximal~的优化是：
select columnList from table where Filter;
或者~table sampling~。

不足：TBL\%SEL\%SEL\%其实是可以优化的。
select key from (select * from src) t1 limit 1;
\subsection{谓词下推}

\subsection{~GroupBy~的实现}
\subsection{join~}

\subsection{union~}
\subsection{分区裁减优化}
PartitionConditionRemover
规则匹配：
(TS\%FIL\%)|(TS\%FIL\%FIL\%)

缺陷：如果包含~Later View~这种语法，后面包含~Partition Cond and NonDeterminst Cond, 将导致分区裁减优化失效。

\section{物理计划生成}
genMapRedTasks
\iffalse
在p-api中，如果要完成根据索引文件大小动态决定模式，如果通过插入Task的方式，由于根据Task的结果动态决定后续该启动的Task，这个hive目前不好支持，因为hive在genMapRedTasks中生成MapReduce任务是这么实现的：
1、	判断是否需要MapReduce（如limit语句可能优化后变成一个FetchTask搞定）；
2、	将op树划分，生成所有的tasks；
3、	进行物理优化；
4、	决定执行模式，这一步中将取出所有的mapreduce任务，逐个判断任务是否需要采用本地执行，如果可本地执行，就设置localMode(true)；这里是调用work的getInputSummary来计算输入大小的。如果partition实现ContentSummaryInputFormat，就可以重载计算其实际的大小。
而Task的执行plan其实在compile时完成，而不在exec中动态决定。
\fi

规则主要包括：
\begin{lstlisting}[language=JAVA]
opRules.put(new RuleRegExp(new String("R1"), "TS%"), new GenMRTableScan1());
opRules.put(new RuleRegExp(new String("R2"), "TS%.*RS%"), new GenMRRedSink1());
opRules.put(new RuleRegExp(new String("R3"), "RS%.*RS%"), new GenMRRedSink2());
opRules.put(new RuleRegExp(new String("R4"), "FS%"), new GenMRFileSink1());
opRules.put(new RuleRegExp(new String("R5"), "UNION%"), new GenMRUnion1());
opRules.put(new RuleRegExp(new String("R6"), "UNION%.*RS%"), new GenMRRedSink3());
opRules.put(new RuleRegExp(new String("R6"), "MAPJOIN%.*RS%"), new GenMRRedSink4());//R6-2
opRules.put(new RuleRegExp(new String("R7"), "TS%.*MAPJOIN%"), MapJoinFactory.getTableScanMapJoin());
opRules.put(new RuleRegExp(new String("R8"), "RS%.*MAPJOIN%"), MapJoinFactory.getReduceSinkMapJoin());
opRules.put(new RuleRegExp(new String("R9"), "UNION%.*MAPJOIN%"), MapJoinFactory.getUnionMapJoin());
opRules.put(new RuleRegExp(new String("R10"), "MAPJOIN%.*MAPJOIN%"), MapJoinFactory.getMapJoinMapJoin());
opRules.put(new RuleRegExp(new String("R11"), "MAPJOIN%SEL%"), MapJoinFactory.getMapJoin());

但是在开源~HIVE~的最新实现中进行了优化，合并对MapJoin的优化，即仅包含：
R1~R6相同(不含~R6-2)，最后添加了~R7~: "MAPJOIN%", MapJoinFactory.getTableScanMapJoin()

\end{lstlisting}

\subsubsection{R1 TS\%}

TS\% 构造一个新的MapRedTask，并在ctx中记录该task为currTask，记录当前operator为currTopOp.
如果是~analyze command，则相对复杂一些。

\begin{lstlisting}[language=JAVA]
public static void setTaskPlan(String alias_id, Operator<? extends Serializable> topOp, MapredWork plan, boolean local, GenMRProcContext opProcCtx, PrunedPartitionList pList) {
  //设置~plan~的~PathToAliases, PathToPartitionInfo~， opToPartList 等信息。
  //为此，获取到~PrunedPartitionList,会调用~PartitionPruner.prune~! ？为什么不让~PCR~先优化再做这个？
  plan.getPathToAliases().get(path).add(alias_id);
  plan.getPathToPartitionInfo().put(path, prtDesc);

  //另外，这里会尝试做一个优化，如果之前~GlobalLimit~优化成功，意味着
  //no aggregation, group-by, distinct, sort by,distributed by, or table sampling in any of the sub-query.
  //如果第一个分区的大小满足要求，就优化~Path，如果文件数目超过限制，或者~SamplePrune~启用,又会禁用这个优化
  //！TODO 这个逻辑比较诡异，待进一步确认
}

\end{lstlisting}
在parseCtx中找到当前operator对应的alias；
为当前operator构造一个GenMapRedCtx对象，以currTask，currTopOp和alias为参数，并以当前operator为key记录在ctx中。

//ANALYZE TABLE T [PARTITION (...)] COMPUTE STATISTICS;
// The plan consists of a simple MapRedTask followed by a StatsTask.
// The MR task is just a simple TableScanOperator

select a, b from src;

select a from (select a, b from src);

select count(distinct key), sum(key) from src group by value;

UNION ALL:

select * from (select key, value from src union all select k as key, v as value from kv1) tmp;

JOIN:
select * from src join kv1 on src1.key=kv1.k;


TS-0 -> SEL-1\\
               UNION-4 -> SEL-5 -> FS-6
TS-2 -> SEL-3/


\subsection{R2 TS\%.*RS\%}

explain select count(*) from src;

GenMRRedSink1
\begin{lstlisting}
    opMapTask = RS~节点第~0~个孩子节点对应的~Task；
    // If the plan for this reducer does not exist, initialize the plan
    if (opMapTask == null) { //如果RS~节点的孩子Operator没有绑定Task
      if (currPlan.getReducer() == null) { //如果当前task还没有reduce，则作为一轮MR任务的reduce task
        GenMapRedUtils.initPlan(op, ctx);
      } else { //新生成一个~MR~任务
        GenMapRedUtils.splitPlan(op, ctx);
      }
    } else {
      // This will happen in case of joins. 
      // The current plan can be thrown away
      // after being merged with the original plan
      GenMapRedUtils.joinPlan(op, null, opMapTask, ctx, -1, false, false, false);
      currTask = opMapTask;
      ctx.setCurrTask(currTask);
    }
\end{lstlisting}

如果当前RS的子operator没有绑定task，且当前task没有设置reducer的时候，那么会将这个RS的子operator设为当前task的reducer。比如：
select count(*) from src;


如果当前RS的子operator（reducer逻辑）只能放到一个新的task（暂时称子task）里去完成了。

所以这个splitPlan方法会生成一个新的plan，并生成一个临时文件信息，通过这个临时文件将父task中的处理结果传递到子task中，由当前RS的子operator（新plan中的reducer）继续执行。这里还需要将当前RS（在父task中）替换为FS，而这个RS会被移到新创建的plan的map部分，当然它不能以top operator的形式出现，所以还会为它创建一个父operator——TS。比如：

对于~JOIN~而言，两个~RS~节点的孩子节点都指向~JOIN~节点，因此第一个~RS~节点处理时，join算子对应的~Task~就非空了。
此时会执行合并操作。
joinPlan的逻辑会通过splitPlan方法为当前RS新建一个plan，并把这个新的plan与子operator对应的task合并为一个task。

一个~TS~有多个~RS~节点


explain from kv1 insert overwrite table src select k key, count(*) as value group by k insert overwrite table kv1 select k, count(*) v1 group by k;

ELSE分支，

explain select key, value from src join kv1 on src.key=kv1.k;

如果是join情况，当前RS的子operator可能已经有对应的task信息了（它已经在join的另一路中被遍历过了），所以此刻会将当前的plan与RS的子operator对应的plan相合并，并将当前plan及task对象抛弃。

\subsection{RS\%.*RS\%}
  GenMRRedSink2
  explain select key, count(*) from src join kv1 on src.key=kv1.k group by key;
  显然在匹配到这个规则，说明当前的节点肯定至少是第二个~RS~，因此之前必然有一个~Reducer，于是必须~splitPlan~

\section{物理优化器}

每个优化器实现接口如下：
\begin{lstlisting}
public interface PhysicalPlanResolver {
  PhysicalContext resolve(PhysicalContext pctx) throws SemanticException;
}
\end{lstlisting}

调用PhysicalOptimizer的optimize时，将遍历所有的~PhysicalPlanResolver~的~resolve~方法。是否启用优化器可以配置，不过~BackendResolver总是加入。BackendResolver~它在resolve中将调用tbl对应storagehandler的getPhysicalPlanResolver进行resolver。

目前PhysicalPlanResolver包括：

\subsection{CommonJoinResolver}

\subsection{MapJoinResolver}

\subsection{SkewJoinResolver}

\subsection{IndexWhereResolver}

\subsection{MetadataOnlyOptimizer}
对于只涉及~Meta~的查询，如：
~select max(ds) from TEST1; 
select ds, count(hr) from TEST2 group by ds;~
其中~ds, hr~是~Partition~列。

%\begin{advanced}\small
%\danger\\hive.input.format~必须是~CombineHiveInputFormat，否则报错。因为~filesplit~的~path~构造~Path~不允许为空\enddanger
%\end{advanced}

从而计算~split~等都可以省略。如~select 

\subsection{BackendResolver}
这个会调用所有后端的优化器。例如~ddbs~优化器。
存在的问题：目前的resolver可能影响其他后端的实现。因为resolve总是会被调用，而不论是否和这个后端有无关系。


\subsection{~MapJoin~优化}

\section{~HIVE~数据模型}

原子类型：
复杂类型：
Map/Struct/Array

\subsection{Object \& ObjectInspector}

\section{执行器}

\subsection{MapOperator}


\subsection{求值树}
Operator { Evaluator[] }
.processOp{ evaluator.evaluate() }

\subsection{JOIN的实现}

\subsection{FileSinkOperator}

createBucketFiles

有些地方insert overwrite directory, newscratchdir

\section{UDF}
\subsection{UDF}
UDF \& GenericUDF
\subsection{UDAF}
\subsection{UDTF}

