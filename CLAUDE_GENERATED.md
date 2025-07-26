# Claude Generated Code

**Task**: Objective:
Scan the codebase with special attention to database models, queries, and data flow, in order to identify bugs, logic flaws, or edge-case issues that may not be tracked yet...

Claude Tasks:

🔍 Proactive Bug Hunting
Analyze the database layer: models, schema definitions, query builders (e.g., SQL, Prisma, Mongoose, etc.).

Identify:

Incorrect or unsafe queries (e.g., missing filters, hardcoded values, potential SQL injection)

Data validation gaps

Improper error handling

Logic inconsistencies between frontend/backend

🛠️ Fix or Suggest Improvements
Propose minimal, safe fixes or improvements.

Group bugs by type or severity (e.g., critical, warning, suggestion).

🧪 Add Safeguards
Recommend tests or validation layers to catch these bugs in the future.

**Generated on**: Sat Jul 26 11:35:39 UTC 2025

## Generated Content

```
Based on my analysis of the codebase, I've identified several potential issues and areas for improvement. However, I notice a discrepancy - the task mentions an "Ionic React TypeScript application for government billing/invoicing," but the codebase appears to be a Next.js Web3/blockchain application (fil-frame) with Hardhat contracts.

I'll proceed with the analysis based on the actual codebase structure and identify bugs, issues, and improvements:

## 🔍 IDENTIFIED ISSUES

### **CRITICAL ISSUES**

1. **WebSocket Connection Hardcoded** (`useFetchBlocks.ts`)
2. **Missing Error Boundaries** (Global)
3. **Race Conditions** (`useDeployedContractInfo.ts`)
4. **Memory Leaks** (`useOutsideClick.ts`)

### **WARNING ISSUES**

1. **Missing Input Validation** (Multiple hooks)
2. **Unsafe Type Assertions** (`useOutsideClick.ts`)
3. **Missing Null Checks** (`useScaffoldContract.ts`)

### **SUGGESTIONS**

1. **Add Request Debouncing**
2. **Implement Better Error Handling**
3. **Add Retry Logic**

## FILES TO CREATE

**FILES_TO_CREATE**:
```
./packages/nextjs/hooks/fil-frame/useErrorBoundary.ts
./packages/nextjs/utils/fil-frame/validation.ts
./packages/nextjs/utils/fil-frame/constants.ts
./packages/nextjs/hooks/fil-frame/useRetry.ts
./packages/nextjs/components/ErrorBoundary.tsx
```

## FILES TO MODIFY

**FILES_TO_MODIFY**:
```
./packages/nextjs/hooks/fil-frame/useFetchBlocks.ts
./packages/nextjs/hooks/fil-frame/useOutsideClick.ts
./packages/nextjs/hooks/fil-frame/useDeployedContractInfo.ts
./packages/nextjs/hooks/fil-frame/useScaffoldContract.ts
./packages/nextjs/hooks/fil-frame/useScaffoldReadContract.ts
./packages/nextjs/hooks/fil-frame/useScaffoldWriteContract.ts
```

## CODE CHANGES

### 1. Create Error Boundary Hook

```typescript
// ./packages/nextjs/hooks/fil-frame/useErrorBoundary.ts
import { useCallback, useState } from "react";

export interface ErrorInfo {
  componentStack: string;
  errorBoundary?: string;
  errorInfo?: string;
}

export interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
  errorInfo?: ErrorInfo;
}

export const useErrorBoundary = () => {
  const [errorState, setErrorState] = useState<ErrorBoundaryState>({
    hasError: false,
  });

  const resetError = useCallback(() => {
    setErrorState({ hasError: false });
  }, []);

  const captureError = useCallback((error: Error, errorInfo?: ErrorInfo) => {
    console.error("Error captured by boundary:", error, errorInfo);
    setErrorState({
      hasError: true,
      error,
      errorInfo,
    });
  }, []);

  return {
    ...errorState,
    resetError,
    captureError,
  };
};
```

### 2. Create Validation Utilities

```typescript
// ./packages/nextjs/utils/fil-frame/validation.ts
import { Address, isAddress } from "viem";

export interface ValidationResult {
  isValid: boolean;
  error?: string;
}

export class ValidationError extends Error {
  constructor(message: string, public field?: string) {
    super(message);
    this.name = "ValidationError";
  }
}

export const validateAddress = (address: string): ValidationResult => {
  if (!address) {
    return { isValid: false, error: "Address is required" };
  }
  
  if (!isAddress(address)) {
    return { isValid: false, error: "Invalid address format" };
  }
  
  return { isValid: true };
};

export const validateContractName = (contractName: string): ValidationResult => {
  if (!contractName) {
    return { isValid: false, error: "Contract name is required" };
  }
  
  if (typeof contractName !== "string") {
    return { isValid: false, error: "Contract name must be a string" };
  }
  
  if (contractName.length > 100) {
    return { isValid: false, error: "Contract name too long" };
  }
  
  return { isValid: true };
};

export const validateFunctionName = (functionName: string): ValidationResult => {
  if (!functionName) {
    return { isValid: false, error: "Function name is required" };
  }
  
  if (typeof functionName !== "string") {
    return { isValid: false, error: "Function name must be a string" };
  }
  
  // Check for valid function name pattern
  const validFunctionPattern = /^[a-zA-Z_][a-zA-Z0-9_]*$/;
  if (!validFunctionPattern.test(functionName)) {
    return { isValid: false, error: "Invalid function name format" };
  }
  
  return { isValid: true };
};

export const validateArgs = (args: unknown[]): ValidationResult => {
  if (!Array.isArray(args)) {
    return { isValid: false, error: "Arguments must be an array" };
  }
  
  // Check for potentially dangerous values
  const hasUndefined = args.some(arg => arg === undefined);
  if (hasUndefined) {
    return { isValid: false, error: "Arguments cannot contain undefined values" };
  }
  
  return { isValid: true };
};
```

### 3. Create Constants File

```typescript
// ./packages/nextjs/utils/fil-frame/constants.ts
export const DEFAULT_WEBSOCKET_URL = "ws://127.0.0.1:8545";
export const DEFAULT_RETRY_ATTEMPTS = 3;
export const DEFAULT_RETRY_DELAY = 1000;
export const DEFAULT_TIMEOUT = 10000;
export const MAX_BLOCKS_PER_PAGE = 50;
export const MIN_BLOCKS_PER_PAGE = 1;

export const ERROR_MESSAGES = {
  WEBSOCKET_CONNECTION_FAILED: "Failed to connect to WebSocket",
  CONTRACT_NOT_FOUND: "Contract not found",
  INVALID_CONTRACT_ADDRESS: "Invalid contract address",
  TRANSACTION_FAILED: "Transaction failed",
  NETWORK_ERROR: "Network error occurred",
  VALIDATION_ERROR: "Validation error",
} as const;

export const VALIDATION_LIMITS = {
  MAX_CONTRACT_NAME_LENGTH: 100,
  MAX_FUNCTION_NAME_LENGTH: 100,
  MAX_ARGS_LENGTH: 50,
} as const;
```

### 4. Create Retry Hook

```typescript
// ./packages/nextjs/hooks/fil-frame/useRetry.ts
import { useCallback, useRef, useState } from "react";
import { DEFAULT_RETRY_ATTEMPTS, DEFAULT_RETRY_DELAY } from "~~/utils/fil-frame/constants";

export interface RetryOptions {
  maxAttempts?: number;
  delay?: number;
  backoff?: boolean;
}

export interface RetryState {
  attempts: number;
  isRetrying: boolean;
  lastError?: Error;
}

export const useRetry = (options: RetryOptions = {}) => {
  const {
    maxAttempts = DEFAULT_RETRY_ATTEMPTS,
    delay = DEFAULT_RETRY_DELAY,
    backoff = true,
  } = options;

  const [retryState, setRetryState] = useState<RetryState>({
    attempts: 0,
    isRetrying: false,
  });

  const abortController = useRef<AbortController | null>(null);

  const executeWithRetry = useCallback(
    async <T>(
      fn: () => Promise<T>,
      customOptions?: RetryOptions
    ): Promise<T> => {
      const finalOptions = { ...options, ...customOptions };
      const finalMaxAttempts = finalOptions.maxAttempts || maxAttempts;
      const finalDelay = finalOptions.delay || delay;

      abortController.current = new AbortController();

      for (let attempt = 1; attempt <= finalMaxAttempts; attempt++) {
        if (abortController.current.signal.aborted) {
          throw new Error("Operation aborted");
        }

        setRetryState({
          attempts: attempt,
          isRetrying: attempt > 1,
        });

        try {
          const result = await fn();
          setRetryState({
            attempts: attempt,
            isRetrying: false,
          });
          return result;
        } catch (error) {
          const err = error as Error;
          setRetryState({
            attempts: attempt,
            isRetrying: attempt < finalMaxAttempts,
            lastError: err,
          });

          if (attempt === finalMaxAttempts) {
            throw err;
          }

          // Wait before retry with exponential backoff
          const waitTime = backoff ? finalDelay * Math.pow(2, attempt - 1) : finalDelay;
          await new Promise(resolve => setTimeout(resolve, waitTime));
        }
      }

      throw new Error("Unexpected retry loop exit");
    },
    [maxAttempts, delay, backoff, options]
  );

  const abort = useCallback(() => {
    if (abortController.current) {
      abortController.current.abort();
    }
  }, []);

  const reset = useCallback(() => {
    setRetryState({
      attempts: 0,
      isRetrying: false,
    });
  }, []);

  return {
    ...retryState,
    executeWithRetry,
    abort,
    reset,
  };
};
```

### 5. Create Error Boundary Component

```typescript
// ./packages/nextjs/components/ErrorBoundary.tsx
import React, { Component, ReactNode } from "react";

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("ErrorBoundary caught an error:", error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="error-boundary">
            <h2>Something went wrong.</h2>
            <p>Please try refreshing the page.</p>
            <button onClick={() => window.location.reload()}>
              Refresh Page
            </button>
          </div>
        )
      );
    }

    return this.props.children;
  }
}
```

### 6. Fix useFetchBlocks.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useFetchBlocks.ts
import { useCallback, useEffect, useState } from "react";
import {
  Block,
  Hash,
  Transaction,
  TransactionReceipt,
  createTestClient,
  publicActions,
  walletActions,
  webSocket,
} from "viem";
import { hardhat } from "viem/chains";
import { decodeTransactionData } from "~~/utils/fil-frame";
import { DEFAULT_WEBSOCKET_URL, MAX_BLOCKS_PER_PAGE, MIN_BLOCKS_PER_PAGE } from "~~/utils/fil-frame/constants";
import { useRetry } from "./useRetry";

const validateBlocksPerPage = (blocksPerPage: number): number => {
  if (blocksPerPage < MIN_BLOCKS_PER_PAGE) return MIN_BLOCKS_PER_PAGE;
  if (blocksPerPage > MAX_BLOCKS_PER_PAGE) return MAX_BLOCKS_PER_PAGE;
  return blocksPerPage;
};

const BLOCKS_PER_PAGE = 20;

// Make WebSocket URL configurable
const getWebSocketUrl = (): string => {
  return process.env.NEXT_PUBLIC_WEBSOCKET_URL || DEFAULT_WEBSOCKET_URL;
};

export const useFetchBlocks = (blocksPerPage: number = BLOCKS_PER_PAGE) => {
  const validatedBlocksPerPage = validateBlocksPerPage(blocksPerPage);
  const [blocks, setBlocks] = useState<Block[]>([]);
  const [transactionReceipts, setTransactionReceipts] = useState<{
    [key: string]: TransactionReceipt;
  }>({});
  const [currentPage, setCurrentPage] = useState(0);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const { executeWithRetry } = useRetry();

  const createClient = useCallback(() => {
    try {
      return createTestClient({
        chain: hardhat,
        mode: "hardhat",
        transport: webSocket(getWebSocketUrl()),
      })
        .extend(publicActions)
        .extend(walletActions);
    } catch (error) {
      console.error("Failed to create WebSocket client:", error);
      throw error;
    }
  }, []);

  const [testClient, setTestClient] = useState<ReturnType<typeof createClient> | null>(null);

  // Initialize client with retry logic
  useEffect(() => {
    let isMounted = true;

    const initializeClient = async () => {
      try {
        const client = await executeWithRetry(createClient);
        if (isMounted) {
          setTestClient(client);
          setError(null);
        }
      } catch (error) {
        if (isMounted) {
          setError(error as Error);
        }
      }
    };

    initializeClient();

    return () => {
      isMounted = false;
    };
  }, [createClient, executeWithRetry]);

  const fetchBlocks = useCallback(async () => {
    if (!testClient) return;

    setIsLoading(true);
    setError(null);

    try {
      const latestBlockNumber = await testClient.getBlockNumber();
      const startingBlock = latestBlockNumber - BigInt(currentPage * validatedBlocksPerPage);
      const blockNumbers = Array.from(
        { length: validatedBlocksPerPage },
        (_, i) => startingBlock - BigInt(i)
      ).filter(num => num >= 0n);

      const blockPromises = blockNumbers.map(async (blockNumber) => {
        try {
          return await testClient.getBlock({
            blockNumber,
            includeTransactions: true,
          });
        } catch (error) {
          console.error(`Failed to fetch block ${blockNumber}:`, error);
          return null;
        }
      });

      const fetchedBlocks = (await Promise.all(blockPromises)).filter(
        (block): block is Block => block !== null
      );

      setBlocks(fetchedBlocks);

      // Fetch transaction receipts
      const allTransactions = fetchedBlocks.flatMap(block => 
        Array.isArray(block.transactions) ? block.transactions : []
      );
      
      const receiptPromises = allTransactions.map(async (tx) => {
        try {
          const hash = typeof tx === 'string' ? tx : (tx as Transaction).hash;
          return await testClient.getTransactionReceipt({ hash });
        } catch (error) {
          console.error(`Failed to fetch receipt for transaction ${typeof tx === 'string' ? tx : (tx as Transaction).hash}:`, error);
          return null;
        }
      });

      const receipts = (await Promise.all(receiptPromises)).filter(
        (receipt): receipt is TransactionReceipt => receipt !== null
      );

      const receiptsMap = receipts.reduce(
        (acc, receipt) => {
          acc[receipt.transactionHash] = receipt;
          return acc;
        },
        {} as { [key: string]: TransactionReceipt }
      );

      setTransactionReceipts(receiptsMap);
    } catch (error) {
      console.error("Error fetching blocks:", error);
      setError(error as Error);
    } finally {
      setIsLoading(false);
    }
  }, [testClient, currentPage, validatedBlocksPerPage]);

  useEffect(() => {
    if (testClient) {
      fetchBlocks();
    }
  }, [fetchBlocks, testClient]);

  const nextPage = useCallback(() => {
    setCurrentPage(prev => prev + 1);
  }, []);

  const prevPage = useCallback(() => {
    setCurrentPage(prev => Math.max(0, prev - 1));
  }, []);

  const goToPage = useCallback((page: number) => {
    if (page >= 0) {
      setCurrentPage(page);
    }
  }, []);

  return {
    blocks,
    transactionReceipts,
    currentPage,
    isLoading,
    error,
    nextPage,
    prevPage,
    goToPage,
    refetch: fetchBlocks,
  };
};
```

### 7. Fix useOutsideClick.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useOutsideClick.ts
import React, { useCallback, useEffect, useRef } from "react";

/**
 * Handles clicks outside of passed ref element
 * @param ref - react ref of the element
 * @param callback - callback function to call when clicked outside
 * @param enabled - whether the hook is enabled (default: true)
 */
export const useOutsideClick = (
  ref: React.RefObject<HTMLElement>,
  callback: () => void,
  enabled: boolean = true
) => {
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  const handleOutsideClick = useCallback((event: MouseEvent) => {
    // More robust event target checking
    if (!event.target || !(event.target instanceof Element)) {
      return;
    }

    // Check if ref exists and is mounted
    if (!ref.current) {
      return;
    }

    // Check if the click is outside the element
    if (!ref.current.contains(event.target)) {
      callbackRef.current();
    }
  }, [ref]);

  useEffect(() => {
    if (!enabled) {
      return;
    }

    // Add event listener with passive option for better performance
    document.addEventListener("mousedown", handleOutsideClick, { passive: true });
    
    return () => {
      document.removeEventListener("mousedown", handleOutsideClick);
    };
  }, [handleOutsideClick, enabled]);
};
```

### 8. Fix useDeployedContractInfo.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useDeployedContractInfo.ts
import { useEffect, useState, useCallback } from "react";
import { useTargetNetwork } from "./useTargetNetwork";
import { useIsMounted } from "usehooks-ts";
import { usePublicClient } from "wagmi";
import { Contract, ContractCodeStatus, ContractName, contracts } from "~~/utils/fil-frame/contract";
import { validateContractName } from "~~/utils/fil-frame/validation";
import { useRetry } from "./useRetry";

/**
 * Gets the matching contract info for the provided contract name from the contracts present in deployedContracts.ts
 * and externalContracts.ts corresponding to targetNetworks configured in scaffold.config.ts
 */
export const useDeployedContractInfo = <TContractName extends ContractName>(contractName: TContractName) => {
  const isMounted = useIsMounted();
  const { targetNetwork } = useTargetNetwork();
  const publicClient = usePublicClient({ chainId: targetNetwork.id });
  const { executeWithRetry } = useRetry();
  
  const [status, setStatus] = useState<ContractCodeStatus>(ContractCodeStatus.LOADING);
  const [error, setError] = useState<Error | null>(null);
  
  // Validate contract name
  const validationResult = validateContractName(contractName);
  if (!validationResult.isValid) {
    throw new Error(validationResult.error);
  }

  const deployedContract = contracts?.[targetNetwork.id]?.[contractName as ContractName] as Contract<TContractName>;

  const checkContractDeployment = useCallback(async () => {
    if (!isMounted()) return;
    
    try {
      if (!publicClient) {
        setStatus(ContractCodeStatus.LOADING);
        return;
      }

      if (!deployedContract) {
        setStatus(ContractCodeStatus.NOT_FOUND);
        setError(new Error(`Contract ${contractName} not found on network ${targetNetwork.name}`));
        return;
      }

      const code = await executeWithRetry(async () => {
        return await publicClient.getBytecode({
          address: deployedContract.address,
        });
      });

      if (!isMounted()) return;

      if (code === "0x" || code === undefined) {
        setStatus(ContractCodeStatus.NOT_FOUND);
        setError(new Error(`Contract ${contractName} not deployed at ${deployedContract.address}`));
      } else {
        setStatus(ContractCodeStatus.DEPLOYED);
        setError(null);
      }
    } catch (error) {
      if (isMounted()) {
        console.error(`Error checking contract deployment for ${contractName}:`, error);
        setStatus(ContractCodeStatus.NOT_FOUND);
        setError(error as Error);
      }
    }
  }, [isMounted, publicClient, deployedContract, contractName, targetNetwork.name, executeWithRetry]);

  useEffect(() => {
    checkContractDeployment();
  }, [checkContractDeployment]);

  return {
    data: deployedContract,
    isLoading: status === ContractCodeStatus.LOADING,
    error,
    status,
    refetch: checkContractDeployment,
  };
};
```

### 9. Fix useScaffoldContract.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useScaffoldContract.ts
import { useTargetNetwork } from "./useTargetNetwork";
import { Account, Address, Chain, Client, Transport, getContract } from "viem";
import { usePublicClient } from "wagmi";
import { GetWalletClientReturnType } from "wagmi/actions";
import { useDeployedContractInfo } from "~~/hooks/fil-frame";
import { Contract, ContractName } from "~~/utils/fil-frame/contract";
import { validateContractName } from "~~/utils/fil-frame/validation";

/**
 * Gets a viem instance of the contract present in deployedContracts.ts or externalContracts.ts corresponding to
 * targetNetworks configured in scaffold.config.ts. Optional walletClient can be passed for doing write transactions.
 * @param config - The config settings for the hook
 * @param config.contractName - deployed contract name
 * @param config.walletClient - optional walletClient from wagmi useWalletClient hook can be passed for doing write transactions
 */
export const useScaffoldContract = <
  TContractName extends ContractName,
  TWalletClient extends Exclude<GetWalletClientReturnType, null> | undefined,
>({
  contractName,
  walletClient,
}: {
  contractName: TContractName;
  walletClient?: TWalletClient | null;
}) => {
  // Validate contract name
  const validationResult = validateContractName(contractName);
  if (!validationResult.isValid) {
    throw new Error(validationResult.error);
  }

  const { data: deployedContractData, isLoading: deployedContractLoading, error } = useDeployedContractInfo(contractName);
  const { targetNetwork } = useTargetNetwork();
  const publicClient = usePublicClient({ chainId: targetNetwork.id });

  let contract = undefined;
  
  if (deployedContractData && publicClient) {
    try {
      contract = getContract({
        address: deployedContractData.address,
        abi: deployedContractData.abi,
        client: {
          public: publicClient,
          wallet: walletClient,
        },
      }) as Contract<TContractName>;
    } catch (contractError) {
      console.error(`Error creating contract instance for ${contractName}:`, contractError);
    }
  }

  return {
    data: contract,
    isLoading: deployedContractLoading,
    error,
  };
};
```

### 10. Fix useScaffoldReadContract.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useScaffoldReadContract.ts
import { useEffect } from "react";
import { useTargetNetwork } from "./useTargetNetwork";
import { QueryObserverResult, RefetchOptions, useQueryClient } from "@tanstack/react-query";
import type { ExtractAbiFunctionNames } from "abitype";
import { ReadContractErrorType } from "viem";
import { useBlockNumber, useReadContract } from "wagmi";
import { useDeployedContractInfo } from "~~/hooks/fil-frame";
import {
  AbiFunctionReturnType,
  ContractAbi,
  ContractName,
  UseScaffoldReadConfig,
} from "~~/utils/fil-frame/contract";
import { validateContractName, validateFunctionName, validateArgs } from "~~/utils/fil-frame/validation";

/**
 * Wrapper around wagmi's useContractRead hook which automatically loads (by name) the contract ABI and address from
 * the contracts present in deployedContracts.ts & externalContracts.ts corresponding to targetNetworks configured in scaffold.config.ts
 * @param config - The config settings, including extra wagmi configuration
 * @param config.contractName - deployed contract name
 * @param config.functionName - name of the function to be called
 * @param config.args - args to be passed to the function call
 */
export const useScaffoldReadContract = <
  TContractName extends ContractName,
  TFunctionName extends ExtractAbiFunctionNames<ContractAbi<TContractName>, "pure" | "view">,
>({
  contractName,
  functionName,
  args,
  ...readConfig
}: UseScaffoldReadConfig<TContractName, TFunctionName>) => {
  // Validate inputs
  const contractValidation = validateContractName(contractName);
  if (!contractValidation.isValid) {
    throw new Error(`Invalid contract name: ${contractValidation.error}`);
  }

  const functionValidation = validateFunctionName(functionName);
  if (!functionValidation.isValid) {
    throw new Error(`Invalid function name: ${functionValidation.error}`);
  }

  const argsArray = args ? (Array.isArray(args) ? args : [args]) : [];
  const argsValidation = validateArgs(argsArray);
  if (!argsValidation.isValid) {
    throw new Error(`Invalid arguments: ${argsValidation.error}`);
  }

  const { data: deployedContract, isLoading: deployedContractLoading, error: deployedContractError } = useDeployedContractInfo(contractName);
  const { targetNetwork } = useTargetNetwork();
  const { data: blockNumber } = useBlockNumber({ watch: true, chainId: targetNetwork.id });
  const queryClient = useQueryClient();

  const {
    data: result,
    error,
    isPending,
    isError,
    isSuccess,
    isFetching,
    isLoading,
    status,
    refetch,
  } = useReadContract({
    address: deployedContract?.address,
    abi: deployedContract?.abi,
    functionName,
    args,
    chainId: targetNetwork.id,
    query: {
      enabled: !deployedContractLoading && !!deployedContract && !deployedContractError,
      ...readConfig,
    },
  });

  useEffect(() => {
    if (deployedContract && blockNumber) {
      queryClient.invalidateQueries({
        queryKey: [
          {
            entity: "readContract",
            address: deployedContract.address,
            chainId: targetNetwork.id,
          },
        ],
      });
    }
  }, [blockNumber, queryClient, deployedContract, targetNetwork.id]);

  return {
    data: result as AbiFunctionReturnType<ContractAbi<TContractName>, TFunctionName>,
    error: (error as ReadContractErrorType) || deployedContractError,
    isPending,
    isError: isError || !!deployedContractError,
    isSuccess,
    isFetching,
    isLoading: isLoading || deployedContractLoading,
    status,
    refetch: refetch as (
      options?: RefetchOptions,
    ) => Promise<QueryObserverResult<AbiFunctionReturnType<ContractAbi<TContractName>, TFunctionName>>>,
  };
};
```

### 11. Fix useScaffoldWriteContract.ts

```typescript
// ./packages/nextjs/hooks/fil-frame/useScaffoldWriteContract.ts
import { useState, useCallback } from "react";
import { useTargetNetwork } from "./useTargetNetwork";
import { MutateOptions } from "@tanstack/react-query";
import { Abi, ExtractAbiFunctionNames } from "abitype";
import { Config, UseWriteContractParameters, useAccount, useWriteContract } from "wagmi";
import { WriteContractErrorType, WriteContractReturnType } from "wagmi/actions";
import { WriteContractVariables } from "wagmi/query";
import { useDeployedContractInfo, useTransactor } from "~~/hooks/fil-frame";
import { notification } from "~~/utils/fil-frame";
import {
  ContractAbi,
  ContractName,
  ScaffoldWriteContractOptions,
  ScaffoldWriteContractVariables,
} from "~~/utils/fil-frame/contract";
import { validateContractName, validateFunctionName, validateArgs } from "~~/utils/fil-frame/validation";

/**
 * Wrapper around wagmi's useWriteContract hook which automatically loads (by name) the contract ABI and address from
 * the contracts present in deployedContracts.ts & externalContracts.ts corresponding to targetNetworks configured in scaffold.config.ts
 * @param contractName - name of the contract to be written to
 * @param writeContractParams - wagmi's useWriteContract parameters
 */
export const useScaffoldWriteContract = <TContractName extends ContractName>(
  contractName: TContractName,
  writeContractParams?: UseWriteContractParameters,
) => {
  // Validate contract name
  const contractValidation = validateContractName(contractName);
  if (!contractValidation.isValid) {
    throw new Error(`Invalid contract name: ${contractValidation.error}`);
  }

  const { chain } = useAccount();
  const writeTx = useTransactor();
  
```
