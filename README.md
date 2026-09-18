/*
Nathan Sanchez
This program creates an outbreak condition and allows the user to input
patient data. It then prints out the priority queue of people based on
personal and regional risk factors. It also allows vaccine distribution
by transferring vaccines to and from connected regions.
*/

#include <iostream>
#include <vector>
#include <map>
#include <queue>
#include <string>
#include <cstdlib>
#include <ctime>
#include <limits>

using namespace std;

int readInt(string prompt)
{
    int value;
    while (true)
    {
        cout << prompt;
        if (cin >> value)
        {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }

        cout << "Please enter a valid number." << endl;
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

int readInt(int minValue, int maxValue, string prompt)
{
    int value;
    while (true)
    {
        cout << prompt;
        if (cin >> value && value >= minValue && value <= maxValue)
        {
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            return value;
        }

        cout << "Please enter a number from "
             << minValue << " to " << maxValue << "." << endl;
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
}

string readLine(string prompt)
{
    string value;
    cout << prompt;
    getline(cin, value);
    return value;
}

int randInt(int minValue, int maxValue)
{
    return minValue + rand() % (maxValue - minValue + 1);
}

// Store values for people
struct Person
{
    string name;
    int age;
    bool immunocompromised;
    bool essentialWorker;
    int exposureLevel;
    int riskScore;
};

// Store values for regions
struct Region
{
    string name;
    int vaccineSupply;
    int outbreakLevel;
    vector<Person> residents;
};

// Compare structs based on riskScore
struct CompareRisk
{
    bool operator()(Person a, Person b)
    {
        return a.riskScore < b.riskScore;
    }
};

// Adds all the factors that affect the riskScore
int calculateRisk(Person p, int outbreakLevel)
{
    int score = 0;

    if (p.age >= 65)
    {
        score += 40;
    }
    else if (p.age >= 50)
    {
        score += 25;
    }

    if (p.immunocompromised)
    {
        score += 35;
    }

    if (p.essentialWorker)
    {
        score += 15;
    }

    score += p.exposureLevel * 5;
    score += outbreakLevel;

    return score;
}

// Generate random outbreak increase and vaccine loss
void generateCrisis(map<string, Region>& regions)
{
    for (auto& regionPair : regions)
    {
        Region& region = regionPair.second;

        int increase = randInt(5, 25);
        region.outbreakLevel += increase;

        int loss = randInt(0, 10);

        if (region.vaccineSupply - loss >= 0)
        {
            region.vaccineSupply -= loss;
        }

        for (int i = 0; i < (int)region.residents.size(); i++)
        {
            region.residents[i].riskScore =
                calculateRisk(
                    region.residents[i],
                    region.outbreakLevel
                );
        }
    }
}

// Displays the parts of the person struct
void displayPerson(Person p)
{
    cout << "Name: " << p.name << endl;
    cout << "Risk Score: " << p.riskScore << endl;
    cout << "Age: " << p.age << endl;
    cout << "Exposure Level: " << p.exposureLevel << endl;

    if (p.immunocompromised)
    {
        cout << "Immunocompromised: YES" << endl;
    }
    else
    {
        cout << "Immunocompromised: NO" << endl;
    }

    if (p.essentialWorker)
    {
        cout << "Essential Worker: YES" << endl;
    }
    else
    {
        cout << "Essential Worker: NO" << endl;
    }

    cout << endl;
}

// Displays the parts of the region struct
void displayRegion(Region r)
{
    cout << "Region: " << r.name << endl;
    cout << "Vaccines Available: " << r.vaccineSupply << endl;
    cout << "Outbreak Level: " << r.outbreakLevel << endl;
    cout << "Population Stored: " << r.residents.size() << endl;
    cout << endl;
}

// Calls all of the user options and creates the regions
int main()
{
    srand((unsigned int)time(0));

    map<string, vector<string>> worldGraph;

    worldGraph["Austin"] = {"Dallas", "Houston"};
    worldGraph["Dallas"] = {"Houston"};
    worldGraph["Houston"] = {"Austin"};

    map<string, Region> regions;

    Region austin;
    austin.name = "Austin";
    austin.vaccineSupply = 100;
    austin.outbreakLevel = 20;

    Region dallas;
    dallas.name = "Dallas";
    dallas.vaccineSupply = 80;
    dallas.outbreakLevel = 10;

    Region houston;
    houston.name = "Houston";
    houston.vaccineSupply = 50;
    houston.outbreakLevel = 30;

    regions["Austin"] = austin;
    regions["Dallas"] = dallas;
    regions["Houston"] = houston;

    cout << "-REGION CONNECTIONS-" << endl;

    for (auto node : worldGraph)
    {
        cout << node.first << " connected to: ";

        for (string connection : node.second)
        {
            cout << connection << " ";
        }

        cout << endl;
    }

    while (true)
    {
        cout << endl;
        cout << "1. Add Patient" << endl;
        cout << "2. View Regions" << endl;
        cout << "3. Generate Crisis" << endl;
        cout << "4. Show Priority Queue" << endl;
        cout << "5. Transfer Vaccines" << endl;
        cout << "6. Exit" << endl;

        int choice = readInt("Choice: ");

        // Adds patients and puts them into regions
        if (choice == 1)
        {
            Person newPerson;
            string regionName;

            newPerson.name = readLine("Enter patient name: ");
            newPerson.age = readInt("Enter age: ");

            int immune =
                readInt(0, 1,
                        "Immunocompromised? (1=yes 0=no): ");

            int worker =
                readInt(0, 1,
                        "Essential worker? (1=yes 0=no): ");

            newPerson.exposureLevel =
                readInt(1, 10,
                        "Exposure level (1-10): ");

            newPerson.immunocompromised = immune;
            newPerson.essentialWorker = worker;

            regionName = readLine("Enter region name: ");

            if (regions.count(regionName))
            {
                newPerson.riskScore =
                    calculateRisk(
                        newPerson,
                        regions[regionName].outbreakLevel
                    );

                regions[regionName].residents.push_back(newPerson);

                cout << "\nPatient added." << endl;
            }
            else
            {
                cout << "\nRegion not found." << endl;
            }
        }

        // Calls displayRegion
        else if (choice == 2)
        {
            cout << "\n-REGIONS-" << endl;

            for (auto regionPair : regions)
            {
                displayRegion(regionPair.second);
            }
        }

        // Calls generateCrisis
        else if (choice == 3)
        {
            generateCrisis(regions);

            cout << "\nCrisis generated." << endl;
        }

        // Creates a priority queue and puts people in order of risk factors
        else if (choice == 4)
        {
            priority_queue<Person, vector<Person>, CompareRisk> vaccineQueue;

            for (auto regionPair : regions)
            {
                Region currentRegion = regionPair.second;

                for (Person p : currentRegion.residents)
                {
                    vaccineQueue.push(p);
                }
            }

            cout << "\n-VACCINE PRIORITY-" << endl;

            while (!vaccineQueue.empty())
            {
                Person highestRisk = vaccineQueue.top();

                displayPerson(highestRisk);

                vaccineQueue.pop();
            }
        }

        // Moves vaccines to and from connected regions
        else if (choice == 5)
        {
            string fromRegion;
            string toRegion;
            int amount;

            cout << "\n-TRANSFER VACCINES-" << endl;

            fromRegion = readLine("Transfer From region: ");
            toRegion = readLine("Transfer To region: ");
            amount = readInt("Number of vaccines: ");

            if (!regions.count(fromRegion) || !regions.count(toRegion))
            {
                cout << "\nInvalid name." << endl;
            }
            else if (amount < 0)
            {
                cout << "\nNumber of vaccines cannot be negative." << endl;
            }
            else
            {
                bool connected = false;

                for (string neighbor : worldGraph[fromRegion])
                {
                    if (neighbor == toRegion)
                    {
                        connected = true;
                    }
                }

                if (!connected)
                {
                    cout << "\nRegions are not connected." << endl;
                }
                else if (regions[fromRegion].vaccineSupply < amount)
                {
                    cout << "\nNot enough vaccines available." << endl;
                }
                else
                {
                    regions[fromRegion].vaccineSupply -= amount;
                    regions[toRegion].vaccineSupply += amount;

                    cout << "\nTransfer successful." << endl;
                    cout << amount << " vaccines moved from "
                         << fromRegion << " to " << toRegion << "."
                         << endl;
                }
            }
        }

        else if (choice == 6)
        {
            break;
        }

        else
        {
            cout << "\nInvalid choice." << endl;
        }
    }

    return 0;
}
