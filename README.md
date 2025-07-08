# AnalyticsProject

Title: Dynamic Pricing for Urban Parking Lots

Organization: Capstone Project, Summer Analytics 2025 (CnA × Pathway)

This project builds a real-time dynamic pricing system for urban parking lots using live data and demand signals such as occupancy, traffic level, queue length, and vehicle type. Implemented entirely with Python, Pandas, Numpy, and Pathway, the system predicts parking prices for 14 lots over 73 days in real time.

**MODEL 1**:
Baseline Linear Model
Pricing Formula
In this simple model, the price at the next time step is linearly dependent on the current occupancy level relative to capacity:
Price at time t+1 = Price at t+𝛼⋅(Occupancy/Capacity)
Where:
α is a constant that controls how sensitively the price responds to occupancy (e.g., α = 2.0)
Initial Price₀ = $10
Prices are bounded between $5 and $20 for realism.

*Rationale*
1)Higher occupancy → higher prices: encourages turnover and optimizes revenue.
2)Lower occupancy → lower prices: attracts more customers when space is underutilized.
3)This basic model ignores traffic, queue length, or vehicle type — it only reacts to how full the parking lot is.

*Assumptions*
1)Occupancy is the only factor influencing price.

2)Parking price updates occur at each time step (~30 min intervals).

3)External factors like traffic or special events are not considered here.

*Limitations*
1)Ignores other demand influencers (queue length, vehicle size, special days).

2)May cause price jumps if occupancy fluctuates rapidly.

3)Acts only as a reference point for comparison with smarter models.

Example Behavior
For alpha = 2, occupancy = 10, capacity = 20, price = 11USD

**MODEL2**:
Demand Function Design
To dynamically adjust the parking price, we modeled demand as a weighted combination of key real-time features:

Demand =𝛼⋅(Occupancy/Capacity)+𝛽⋅QueueLength−𝛾⋅TrafficLevel+𝛿⋅IsSpecialDay+𝜀⋅VehicleTypeWeight
Where:
1)Occupancy / Capacity: Represents current utilization

2)QueueLength: Indicates excess demand

3)TrafficLevel: High congestion can reduce willingness to park

4)IsSpecialDay: Holidays or events increase demand

5)VehicleTypeWeight: Larger vehicles (e.g., trucks) use more space

*Pricing Logic*
We adjusted prices based on the computed demand:

Price at time t = BasePrice⋅(1+𝜆⋅NormalizedDemand)
1)BasePrice = $10

2)λ controls the impact of demand on price

3)Demand was normalized to ensure prices remain bounded:

->Minimum = 0.5 × BasePrice ($5)

->Maximum = 2 × BasePrice ($20)

This ensured prices:

1)Increase smoothly with demand

2)Remain stable and avoid erratic jumps

*Assumptions Made*

1)Queue length reflects unmet demand and willingness to pay more.

2)Special days (festivals, events) increase parking demand.

3)Traffic levels reduce parking appeal — high traffic can discourage parking.

4)Vehicle type impacts price through space usage; for instance:
Cars = 1.0
Bikes = 0.6
Trucks = 1.5

5)Demand function coefficients (α, β, γ, δ, ε) are set heuristically and can be learned from historical data in advanced versions.

Example Outcome
As occupancy and queue length grow, prices gradually increase from $10 to a max of $20. If traffic is high or it's a normal weekday, prices may decrease. The system thus adapts to real-time conditions in a realistic, explainable manner.
The final bokeh graph post model 2 looks like this:
![bokeh_plot](https://github.com/user-attachments/assets/b4ba0ce4-a919-4019-b235-197c15dd1a35)

**Tech Stack**

| Layer                    | Technology                                                                                                     |
| ------------------------ | -------------------------------------------------------------------------------------------------------------- |
| **Language**             | Python 3.11                                                                                                    |
| **Libraries**            | Numpy, Pandas, Scikit-learn (for offline model), Pathway (for real-time simulation), Bokeh (for visualization) |
| **Streaming Engine**     | Pathway                                                                                                        |
| **Development Platform** | Google Colab                                                                                                   |
| **Visualization**        | Bokeh                                                                                                          |
| **Version Control**      | Git & GitHub                                                                                                   |

**Architecture Overview**
1. Offline Training (Model 2)

Load dataset.csv using Pandas

Engineer demand-relevant features

Train ML model (e.g., Random Forest) to predict prices

Save model (e.g., using joblib or pickle)

2. Real-Time Simulation using Pathway
   
Stream dataset.csv with timestamps using pw.io.csv.read

Perform real-time feature engineering (vehicle type, traffic encoding, etc.)

Use pre-trained ML model (or simple formula for Model 1) to predict live prices

Output prediction using pw.io.jsonlines.write

3. Visualization

Live line plots for each parking lot using Bokeh

Compare with competitor prices and base price

**Architechture Flow: Mermaid Diagram**:

flowchart TD
    A[dataset.csv] --> B[Pathway Streaming Engine]

    B --> C[Feature Engineering in Real Time]
    C --> D[Model 1: Linear Pricing]
    C --> E[Model 2: Demand-Based Pricing]

    D --> F[Price Prediction Table]
    E --> F

    F --> G[predicted_prices.jsonl]
    G --> H[Real-Time Bokeh Visualization]

    style A fill:#f9f,stroke:#333,stroke-width:2px
    style B fill:#bbf,stroke:#333,stroke-width:2px
    style C fill:#ddf,stroke:#333,stroke-width:2px
    style D fill:#afa,stroke:#333,stroke-width:2px
    style E fill:#afa,strok


**Architechture Flow Diagram**

            +----------------+
            |  dataset.csv   |
            +--------+-------+
                     |
                     v
           [ Pathway Streaming Engine ]
                     |
        +------------+------------+
        |                         |
        v                         v
[ Feature Engineering ]     [ Pre-trained ML Model ]
        |                         |
        +------------+------------+
                     |
                     v
          [ Price Prediction Table ]
                     |
                     v
         [ Write to predicted_prices.jsonl ]
                     |
                     v
            [ Visualize with Bokeh ]
            


