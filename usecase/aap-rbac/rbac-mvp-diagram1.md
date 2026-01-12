# Diagram RBAC-MVP

```mermaid
graph LR
    subgraph ORG[Organization: RHEL-Operations]
        
        subgraph TEAMS[Teams]
            direction TB
            T1(Team: Platform-Admins)
            T2(Team: RHEL-Specialists)
            T3(Team: RHEL-HelpDesk)
        end

        subgraph ADM_ROLES[Administrative Roles]
            R1[Organization Admin]
        end

        subgraph RES_ROLES[Resource Roles]
            direction TB
            R2[Role: Admin]
            R3[Role: Execute]
        end

        subgraph ASSETS[Resources]
            direction TB
            A1[Structure: Users & Teams]
            J1[Content: Job Templates]
        end

        %% Assignments
        T1 -->|Assigned| R1
        T2 -->|Assigned| R2
        T3 -->|Assigned| R3

        %% Capabilities
        R1 -->|Manage| A1
        R2 -->|Edit/Update/Run| J1
        R3 -->|Run Only| J1
    end
    ```
