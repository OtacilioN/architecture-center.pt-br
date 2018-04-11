---
title: Dados transacionais
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: b7fdbb403d2a438ebc59e40ef58ed8067489dddc
ms.sourcegitcommit: 943e671a8d522cef5ddc8c6e04848134b03c2de4
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 03/05/2018
---
# <a name="transactional-data"></a><span data-ttu-id="03bcb-102">Dados transacionais</span><span class="sxs-lookup"><span data-stu-id="03bcb-102">Transactional data</span></span>

<span data-ttu-id="03bcb-103">Dados transacionais são informações que acompanham as interações relacionadas às atividades de uma organização.</span><span class="sxs-lookup"><span data-stu-id="03bcb-103">Transactional data is information that tracks the interactions related to an organization's activities.</span></span> <span data-ttu-id="03bcb-104">Normalmente, essas interações são transações comerciais, como pagamentos recebidos de clientes, pagamentos feitos a fornecedores e movimentação de produtos pelo inventário, pedidos feitos ou serviços entregues.</span><span class="sxs-lookup"><span data-stu-id="03bcb-104">These interactions are typically business transactions, such as payments received from customers, payments made to suppliers, products moving through inventory, orders taken, or services delivered.</span></span> <span data-ttu-id="03bcb-105">Eventos transacionais, que representam as transações em si, normalmente contêm uma dimensão temporal, alguns valores numéricos e referências a outros dados.</span><span class="sxs-lookup"><span data-stu-id="03bcb-105">Transactional events, which represent the transactions themselves, typically contain a time dimension, some numerical values, and references to other data.</span></span> 

<span data-ttu-id="03bcb-106">As transações normalmente precisam ser *atômicas* e *consistentes*.</span><span class="sxs-lookup"><span data-stu-id="03bcb-106">Transactions typically need to be *atomic* and *consistent*.</span></span> <span data-ttu-id="03bcb-107">Atomicidade significa que uma transação inteira sempre tem êxito ou falha como uma unidade de trabalho e nunca é deixada em um estado concluído incompleto.</span><span class="sxs-lookup"><span data-stu-id="03bcb-107">Atomicity means that an entire transaction always succeeds or fails as one unit of work, and is never left in a half-completed state.</span></span> <span data-ttu-id="03bcb-108">Se uma transação não pode ser concluída, o sistema de banco de dados precisa reverter todas as etapas que já foram realizadas como parte dessa transação.</span><span class="sxs-lookup"><span data-stu-id="03bcb-108">If a transaction cannot be completed, the database system must roll back any steps that were already done as part of that transaction.</span></span> <span data-ttu-id="03bcb-109">Em um RDBMS tradicional, essa reversão ocorre automaticamente se uma transação não pode ser concluída.</span><span class="sxs-lookup"><span data-stu-id="03bcb-109">In a traditional RDBMS, this rollback happens automatically if a transaction cannot be completed.</span></span> <span data-ttu-id="03bcb-110">Consistência significa que as transações sempre deixam os dados em um estado válido.</span><span class="sxs-lookup"><span data-stu-id="03bcb-110">Consistency means that transactions always leave the data in a valid state.</span></span> <span data-ttu-id="03bcb-111">(Essas são descrições muito informais de atomicidade e consistência.</span><span class="sxs-lookup"><span data-stu-id="03bcb-111">(These are very informal descriptions of atomicity and consistency.</span></span> <span data-ttu-id="03bcb-112">Há definições mais formais dessas propriedades, como [ACID](https://en.wikipedia.org/wiki/ACID).)</span><span class="sxs-lookup"><span data-stu-id="03bcb-112">There are more formal definitions of these properties, such as [ACID](https://en.wikipedia.org/wiki/ACID).)</span></span>

<span data-ttu-id="03bcb-113">Bancos de dados transacionais podem dar suporte à coerência forte em transações que usam várias estratégias de bloqueio, como bloqueio pessimista, a fim de garantir que todos os dados sejam altamente consistentes dentro do contexto da empresa, para todos os usuários e processos.</span><span class="sxs-lookup"><span data-stu-id="03bcb-113">Transactional databases can support strong consistency for transactions using various locking strategies, such as pessimistic locking, to ensure that all data is strongly consistent within the context of the enterprise, for all users and processes.</span></span> 

<span data-ttu-id="03bcb-114">A arquitetura de implantação mais comum que usa dados transacionais é a camada de armazenamento de dados em uma arquitetura de três camadas.</span><span class="sxs-lookup"><span data-stu-id="03bcb-114">The most common deployment architecture that uses transactional data is the data store tier in a 3-tier architecture.</span></span> <span data-ttu-id="03bcb-115">Uma arquitetura de três camadas normalmente consiste em uma camada de apresentação, uma camada de lógica de negócios e uma camada de armazenamento de dados.</span><span class="sxs-lookup"><span data-stu-id="03bcb-115">A 3-tier architecture typically consists of a presentation tier, business logic tier, and data store tier.</span></span> <span data-ttu-id="03bcb-116">Uma arquitetura de implantação relacionada é a arquitetura [de N camadas](/azure/architecture/guide/architecture-styles/n-tier), que pode ter várias camadas intermediárias que manipulam a lógica de negócios.</span><span class="sxs-lookup"><span data-stu-id="03bcb-116">A related deployment architecture is the [N-tier](/azure/architecture/guide/architecture-styles/n-tier) architecture, which may have multiple middle-tiers handling business logic.</span></span>

![Exemplo de um aplicativo de três camadas](./images/three-tier-application.png)

## <a name="typical-traits-of-transactional-data"></a><span data-ttu-id="03bcb-118">Características comuns de dados transacionais</span><span class="sxs-lookup"><span data-stu-id="03bcb-118">Typical traits of transactional data</span></span>

<span data-ttu-id="03bcb-119">Os dados transacionais tendem a ter as seguintes características:</span><span class="sxs-lookup"><span data-stu-id="03bcb-119">Transactional data tends to have the following traits:</span></span>

| <span data-ttu-id="03bcb-120">Requisito</span><span class="sxs-lookup"><span data-stu-id="03bcb-120">Requirement</span></span> | <span data-ttu-id="03bcb-121">DESCRIÇÃO</span><span class="sxs-lookup"><span data-stu-id="03bcb-121">Description</span></span> |
| --- | --- |
| <span data-ttu-id="03bcb-122">Normalização</span><span class="sxs-lookup"><span data-stu-id="03bcb-122">Normalization</span></span> | <span data-ttu-id="03bcb-123">Altamente normalizado</span><span class="sxs-lookup"><span data-stu-id="03bcb-123">Highly normalized</span></span> |
| <span data-ttu-id="03bcb-124">Esquema</span><span class="sxs-lookup"><span data-stu-id="03bcb-124">Schema</span></span> | <span data-ttu-id="03bcb-125">Esquema na gravação, altamente imposto</span><span class="sxs-lookup"><span data-stu-id="03bcb-125">Schema on write, strongly enforced</span></span>|
| <span data-ttu-id="03bcb-126">Consistência</span><span class="sxs-lookup"><span data-stu-id="03bcb-126">Consistency</span></span> | <span data-ttu-id="03bcb-127">Coerência forte, garantia de ACID</span><span class="sxs-lookup"><span data-stu-id="03bcb-127">Strong consistency, ACID guarantees</span></span> |
| <span data-ttu-id="03bcb-128">Integridade</span><span class="sxs-lookup"><span data-stu-id="03bcb-128">Integrity</span></span> | <span data-ttu-id="03bcb-129">Alta integridade</span><span class="sxs-lookup"><span data-stu-id="03bcb-129">High integrity</span></span> |
| <span data-ttu-id="03bcb-130">Usa transações</span><span class="sxs-lookup"><span data-stu-id="03bcb-130">Uses transactions</span></span> | <span data-ttu-id="03bcb-131">sim</span><span class="sxs-lookup"><span data-stu-id="03bcb-131">Yes</span></span> |
| <span data-ttu-id="03bcb-132">Estratégia de bloqueio</span><span class="sxs-lookup"><span data-stu-id="03bcb-132">Locking strategy</span></span> | <span data-ttu-id="03bcb-133">Pessimista ou otimista</span><span class="sxs-lookup"><span data-stu-id="03bcb-133">Pessimistic or optimistic</span></span>|
| <span data-ttu-id="03bcb-134">Atualizável</span><span class="sxs-lookup"><span data-stu-id="03bcb-134">Updateable</span></span> | <span data-ttu-id="03bcb-135">sim</span><span class="sxs-lookup"><span data-stu-id="03bcb-135">Yes</span></span> |
| <span data-ttu-id="03bcb-136">Acrescentável</span><span class="sxs-lookup"><span data-stu-id="03bcb-136">Appendable</span></span> | <span data-ttu-id="03bcb-137">sim</span><span class="sxs-lookup"><span data-stu-id="03bcb-137">Yes</span></span> |
| <span data-ttu-id="03bcb-138">Carga de trabalho</span><span class="sxs-lookup"><span data-stu-id="03bcb-138">Workload</span></span> | <span data-ttu-id="03bcb-139">Gravações intensas, leituras moderadas</span><span class="sxs-lookup"><span data-stu-id="03bcb-139">Heavy writes, moderate reads</span></span> |
| <span data-ttu-id="03bcb-140">Indexação</span><span class="sxs-lookup"><span data-stu-id="03bcb-140">Indexing</span></span> | <span data-ttu-id="03bcb-141">Índices primários e secundários</span><span class="sxs-lookup"><span data-stu-id="03bcb-141">Primary and secondary indexes</span></span> |
| <span data-ttu-id="03bcb-142">Tamanho do dado</span><span class="sxs-lookup"><span data-stu-id="03bcb-142">Datum size</span></span> | <span data-ttu-id="03bcb-143">De pequeno a médio</span><span class="sxs-lookup"><span data-stu-id="03bcb-143">Small to medium sized</span></span> |
| <span data-ttu-id="03bcb-144">Modelo</span><span class="sxs-lookup"><span data-stu-id="03bcb-144">Model</span></span> | <span data-ttu-id="03bcb-145">Relacional</span><span class="sxs-lookup"><span data-stu-id="03bcb-145">Relational</span></span> |
| <span data-ttu-id="03bcb-146">Forma dos dados</span><span class="sxs-lookup"><span data-stu-id="03bcb-146">Data shape</span></span> | <span data-ttu-id="03bcb-147">Tabular</span><span class="sxs-lookup"><span data-stu-id="03bcb-147">Tabular</span></span> |
| <span data-ttu-id="03bcb-148">Flexibilidade de consulta</span><span class="sxs-lookup"><span data-stu-id="03bcb-148">Query flexibility</span></span> | <span data-ttu-id="03bcb-149">Altamente flexível</span><span class="sxs-lookup"><span data-stu-id="03bcb-149">Highly flexible</span></span> |
| <span data-ttu-id="03bcb-150">Escala</span><span class="sxs-lookup"><span data-stu-id="03bcb-150">Scale</span></span> | <span data-ttu-id="03bcb-151">Pequeno (MBs) a grande (alguns TBs)</span><span class="sxs-lookup"><span data-stu-id="03bcb-151">Small (MBs) to Large (a few TBs)</span></span> | 

## <a name="see-also"></a><span data-ttu-id="03bcb-152">Veja também</span><span class="sxs-lookup"><span data-stu-id="03bcb-152">See Also</span></span>

[<span data-ttu-id="03bcb-153">Processamento de Transações Online</span><span class="sxs-lookup"><span data-stu-id="03bcb-153">Online Transaction Processing</span></span>](../scenarios/online-transaction-processing.md)