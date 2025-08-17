
"use client";

import { useEffect, useState } from "react";

type OddsData = {
  sport_key: string;
  home_team: string;
  away_team: string;
  bookmakers: {
    title: string;
    markets: {
      key: string;
      outcomes: { name: string; price: number; point?: number }[];
    }[];
  }[];
};

export default function Home() {
  const [odds, setOdds] = useState<OddsData[]>([]);
  const [loading, setLoading] = useState(true);

  const API_KEY = process.env.NEXT_PUBLIC_ODDS_API_KEY; // lấy từ .env

  async function fetchOdds() {
    try {
      const res = await fetch(
        `https://api.the-odds-api.com/v4/sports/soccer/odds/?regions=eu&markets=h2h,spreads,totals&oddsFormat=decimal&apiKey=${API_KEY}`
      );
      const data = await res.json();
      setOdds(data);
    } catch (err) {
      console.error(err);
    } finally {
      setLoading(false);
    }
  }

  // Fetch lần đầu + auto refresh mỗi 30 giây
  useEffect(() => {
    fetchOdds();
    const interval = setInterval(fetchOdds, 30000); // 30 giây
    return () => clearInterval(interval);
  }, []);

  // Hàm tính xác suất từ odds
  function impliedProb(odd: number): string {
    if (!odd) return "-";
    return (100 / odd).toFixed(1) + "%";
  }

  if (loading) return <div className="p-4">⏳ Loading odds...</div>;

  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-6">Live Odds & Probabilities</h1>
      <p className="text-sm text-gray-500 mb-4">
        (Tự động cập nhật mỗi 30 giây)
      </p>
      <table className="min-w-full border border-gray-300 text-sm">
        <thead className="bg-gray-100">
          <tr>
            <th className="border px-2 py-1">Match</th>
            <th className="border px-2 py-1">FT Handicap</th>
            <th className="border px-2 py-1">Prob</th>
            <th className="border px-2 py-1">FT O/U</th>
            <th className="border px-2 py-1">Prob</th>
            <th className="border px-2 py-1">H1 Handicap</th>
            <th className="border px-2 py-1">Prob</th>
            <th className="border px-2 py-1">H1 O/U</th>
            <th className="border px-2 py-1">Prob</th>
          </tr>
        </thead>
        <tbody>
          {odds.map((game, idx) => {
            const bookmaker = game.bookmakers[0]; // lấy 1 nhà cái đầu tiên
            const spreads = bookmaker?.markets.find((m) => m.key === "spreads");
            const totals = bookmaker?.markets.find((m) => m.key === "totals");

            const ftHandicap = spreads?.outcomes[0];
            const ftHandicapProb = spreads?.outcomes[0]
              ? impliedProb(spreads.outcomes[0].price)
              : "-";

            const ftOU = totals?.outcomes[0];
            const ftOUProb = totals?.outcomes[0]
              ? impliedProb(totals.outcomes[0].price)
              : "-";

            // giả lập hiệp 1: nếu API có H1 thì thay bằng dữ liệu thật
            const h1Handicap = spreads?.outcomes[1];
            const h1HandicapProb = spreads?.outcomes[1]
              ? impliedProb(spreads.outcomes[1].price)
              : "-";

            const h1OU = totals?.outcomes[1];
            const h1OUProb = totals?.outcomes[1]
              ? impliedProb(totals.outcomes[1].price)
              : "-";

            return (
              <tr key={idx} className="text-center">
                <td className="border px-2 py-1 font-medium">
                  {game.home_team} vs {game.away_team}
                </td>
                <td className="border px-2 py-1">
                  {ftHandicap?.point ?? "-"} ({ftHandicap?.price ?? "-"})
                </td>
                <td className="border px-2 py-1">{ftHandicapProb}</td>
                <td className="border px-2 py-1">
                  {ftOU?.point ?? "-"} ({ftOU?.price ?? "-"})
                </td>
                <td className="border px-2 py-1">{ftOUProb}</td>
                <td className="border px-2 py-1">
                  {h1Handicap?.point ?? "-"} ({h1Handicap?.price ?? "-"})
                </td>
                <td className="border px-2 py-1">{h1HandicapProb}</td>
                <td className="border px-2 py-1">
                  {h1OU?.point ?? "-"} ({h1OU?.price ?? "-"})
                </td>
                <td className="border px-2 py-1">{h1OUProb}</td>
              </tr>
            );
          })}
        </tbody>
      </table>
    </div>
  );
}
